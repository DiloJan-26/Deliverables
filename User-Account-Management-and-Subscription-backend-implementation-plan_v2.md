# Orbit Scanner — User Account Management & Subscription Backend (Final)

**Your role:** Backend developer + decision-maker for FastAPI. This is your complete spec for identity, the AI-credit system, subscription, and abuse prevention.

**Stack:** FastAPI + uvicorn · Firebase Auth (verify-only) · Firestore · Google Play Billing · Play Integrity API.

**Confirmed model (this version):**
- Unified **"AI credits"** for both Free-Registered and Pro tiers.
- Guest tracks **Extract Text attempts** (5), not credits.
- Internal accounting in **INR (paisa precision)**; UI shows **AI credits**.

> **§0 has two corrections to your model that protect your margin. Read it first.**

---

## §0. Two corrections before anything else

### 0.1 — "Double the rupees to get credits" is backwards. Here's the fix.

Your phrasing: *"₹50/month → double that → 100 credits"* and *"10 credits → 10/2 = ₹5 internally."*

The **outcome** is correct (₹50 ↔ 100 credits, ₹5 ↔ 10 credits), but the **rule** as stated ("double the rupees") will confuse whoever codes it. The clean, unambiguous rule is:

> **1 AI credit = ₹0.50 of internal API budget.**
> Equivalently: **2 AI credits = ₹1.**

Everything derives from that one constant. Never "double" anything in code — just multiply cost-in-INR by 2 to get credits, or credits by 0.5 to get INR. One constant, one direction, no confusion:

```
CREDIT_VALUE_INR = 0.50      # 1 AI credit buys ₹0.50 of API cost

credits  = cost_inr / CREDIT_VALUE_INR      # ₹0.50 cost → 1.0 credit
cost_inr = credits  * CREDIT_VALUE_INR      # 1.0 credit → ₹0.50 cost
```

This single constant produces *exactly* your intended numbers:

| Tier | Credits granted | Internal budget | Verified |
|---|---|---|---|
| Free Registered | 10 credits | ₹5.00 | ✅ |
| Pro (₹99/mo) | 100 credits | ₹50.00 | ✅ |

### 0.2 — Do NOT track credits as the source of truth. Track INR. Show credits.

This is the most important architectural decision in the whole credit system, and it's subtle.

**The problem with storing credits directly:** Gemini bills in USD per token. The USD→INR rate drifts. If you store "user has 43.2 credits" as truth and the exchange rate moves, your credit no longer maps to ₹0.50 of real cost — your margin silently leaks. Also, credits are rounded for display; rounding errors accumulate if credits are the ledger unit.

**The fix — a three-layer model:**

```
┌─────────────────────────────────────────────────────────────┐
│  LAYER 3 — UI (what the user sees)                          │
│  "AI Credits: 87 / 100"  ·  battery bar  ·  percent circle  │
├─────────────────────────────────────────────────────────────┤
│  LAYER 2 — Credit view (derived, never stored as truth)     │
│  credits_remaining = (budget_inr - spent_inr) / 0.50        │
├─────────────────────────────────────────────────────────────┤
│  LAYER 1 — SOURCE OF TRUTH (stored in Firestore)            │
│  budget_inr = 50.00   ·   spent_inr = 6.45  (paisa precise) │
└─────────────────────────────────────────────────────────────┘
```

**You store and reason in INR (paisa-precision, i.e. 4 decimal places).** Credits are *computed on read* for display. This means:
- Exchange-rate drift never corrupts your ledger.
- One unit (INR) for both Sarvam and Gemini — no mixed-unit math.
- The "AI credit" is purely a UI skin over real money. Change the credit ratio later (promo: "double credits this month!") by editing one constant, without touching stored balances.

**This is how every production metered-billing system works** (Twilio, AWS, OpenAI all track money/usage internally and display friendly units). You're doing it right by separating them.

---

## §1. The unit conversion chain (Sarvam & Gemini → INR → credits)

You asked which internal unit is better. **Answer: INR, paisa-precision.** Here's the full conversion for each provider, settled.

### Sarvam Vision (Layout OCR) — simplest
Sarvam bills in *their* credits where 1 Sarvam-credit = ₹1, and Layout OCR = ₹0.50/page.

```
Sarvam cost  →  INR directly
1 page Layout OCR = ₹0.50  →  1.0 AI credit (displayed)
```
No currency conversion. The response includes page count; multiply by ₹0.50.

> **Naming clarity (your concern):** Sarvam's "credits" ≠ your "AI credits." Keep them separate in code:
> - `sarvam_cost_inr` = what Sarvam charges you (₹0.50/page)
> - `ai_credits` = what your user sees (1.0/page)
> Never store or compare them as the same field. Calling yours **"AI Credits"** in the UI (as you suggested) is the right move — zero ambiguity for users.

### Gemini 2.5 Flash-Lite (Scan-to-Word) — needs USD→INR
Gemini bills USD per token. The chain is **USD → INR → (display credits)**, but you **store the INR**, not the credits:

```
Step 1  tokens → USD    : from usageMetadata (promptTokenCount, candidatesTokenCount)
Step 2  USD → INR       : × 83.5  (configurable FX rate)
Step 3  store INR       : add to spent_inr  ← SOURCE OF TRUTH
Step 4  INR → credits   : ÷ 0.50  ← display only, computed on read
```

Verified example (1 page, ~350 in / ~750 out tokens):
```
USD  = (350/1M × $0.10) + (750/1M × $0.40) = $0.000335
INR  = $0.000335 × 83.5                     = ₹0.028
≈ ₹0.03/page  ✓ matches pay_plan
credits = ₹0.028 / 0.50                       = 0.056 ≈ 0.06 credit/page  ✓
```

> **⚠️ Gemini image-token caveat:** Scan-to-Word sends an **image** to Gemini, and Gemini counts image input as a fixed token block (often ~258–1290 tokens/image depending on resolution), *plus* your text prompt, *plus* output. The ₹0.03/page average **must be validated against real `usageMetadata` from actual scans**, not assumed. Log the real token counts for your first 100 jobs and recompute. The *system* doesn't change — only the per-page reality. Budget a safety buffer until you have real data.

### The FX rate — make it config, refresh it
Don't hardcode 83.5. Store `USD_INR_RATE` in Firestore config (or Remote Config). Refresh it weekly from a free FX API (or just review monthly manually for v0 — the rate moves slowly enough). Round costs **up** to stay margin-safe.

---

## §2. Authentication — your job vs frontend's job (settled)

### Is the frontend lead's plan good? **Yes — it's the correct production approach.**

He's right that Google Sign-In belongs on Android via Credential Manager + Google Play Services. Reasons it's better than backend-side OAuth:
- Play Services handles the Google account picker natively (one tap, no webview).
- No redirect-URI juggling, no storing OAuth refresh tokens server-side.
- Works offline-to-online gracefully; Firebase SDK auto-refreshes tokens.
- It's what Google's own docs recommend for Android.

**So the division of labor is:**

```
┌────────────────────────────────────────────────────────────────┐
│  FRONTEND (Android) — owns sign-in                            │
│  • Credential Manager + Sign in with Google                    │
│  • Firebase Auth SDK (email/password + Google)                 │
│  • Holds & auto-refreshes the Firebase ID token                │
│  • Sends ID token in Authorization header on every API call    │
│  • google-services.json lives here (image 2)                   │
├────────────────────────────────────────────────────────────────┤
│  BACKEND (you, FastAPI) — owns verification & state           │
│  • VERIFY the Firebase ID token (Admin SDK)                    │
│  • Create/load the user record in Firestore                    │
│  • Own all tier / credit / subscription / abuse logic          │
│  • Never sees a password, never runs an OAuth redirect         │
│  • Firebase Admin service-account key lives here (you generate)│
└────────────────────────────────────────────────────────────────┘
```

### What you do with the credentials in the images

| Credential | Image | Whose side | Your action |
|---|---|---|---|
| `client_secret.json` ("installed" OAuth client) | Image 1 | — | **Archive. Not needed in v0.** It's for server-side OAuth web flows (e.g. future Drive upload on user's behalf). Keep it secret; don't delete. |
| `google-services.json` (Firebase config) | Image 2 | **Frontend** | Hand to FE lead. Goes in the Android `app/` module. **Not in your backend.** |
| Firebase **Admin SDK** service-account key | — (generate it) | **Backend (you)** | **This is what you actually need.** Generate it, load into FastAPI. |

**Note on image 2:** `project_id: orbit-scanner`, `package_name: com.pluton.orbitscanner`, `storage_bucket: orbit-scanner.firebasestorage.app` are fine to be visible. The **`current_key` / `api_key` you blacked out is a Firebase API key** — restrict it in Google Cloud Console (Application restrictions → Android apps, API restrictions → only the Firebase APIs you use). It's not a deadly secret like a service-account key, but lock it down anyway.

### The one thing you must generate
**Firebase Console → Project Settings → Service Accounts → "Generate new private key."**
That downloads a JSON with a private key. **This** is your backend's credential — it lets FastAPI verify tokens and read/write Firestore as admin. Store it in a secret manager (§7), never in git.

### What you DON'T need to build (so you don't waste time)
- ❌ No password hashing / storage (Firebase does it)
- ❌ No OAuth redirect/callback endpoints
- ❌ No email-verification sending (Firebase SDK does it client-side)
- ❌ No session cookies / server session store (ID token is the session)
- ❌ No "forgot password" flow (Firebase handles it)

### What you DO build
- ✅ Token verification dependency (one function, §2.1)
- ✅ User record creation/sync in Firestore
- ✅ Everything about tiers, credits, subscriptions, abuse

### 2.1 The verification dependency
```python
# app/auth/deps.py
from fastapi import Depends, HTTPException, Header
from firebase_admin import auth as fb_auth

async def get_current_user(authorization: str = Header(None)):
    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(401, "missing_token")
    token = authorization.split(" ", 1)[1]
    try:
        decoded = fb_auth.verify_id_token(token, check_revoked=True)
    except fb_auth.ExpiredIdTokenError:
        raise HTTPException(401, "token_expired")   # app refreshes & retries
    except fb_auth.RevokedIdTokenError:
        raise HTTPException(401, "token_revoked")    # app forces re-login
    except Exception:
        raise HTTPException(401, "invalid_token")
    return await user_repo.get_or_create(decoded["uid"], decoded)
```
That's the entire auth surface. Token expiry (your flagged failure mode) → return `token_expired`; the Firebase SDK on Android refreshes and retries. You stay stateless.

---

## §3. The three tiers & how each is tracked

```
┌──────────────┬─────────────────┬──────────────────────┬─────────────────────┐
│              │ GUEST           │ REGISTERED (free)    │ PRO (₹99/mo)        │
├──────────────┼─────────────────┼──────────────────────┼─────────────────────┤
│ Identity     │ Device + Integ. │ Firebase uid         │ Firebase + Play sub │
│ Extract Text │ 5 attempts      │ Unlimited (on-device)│ Unlimited           │
│   tracked by │ server counter  │ —                    │ —                   │
│ Layout OCR   │ ❌              │ AI credits           │ AI credits          │
│ Scan-to-Word │ ❌              │ AI credits           │ AI credits          │
│ Credit grant │ none            │ 10 (one-time, ₹5)    │ 100/mo (₹50, resets)│
│ Other feats  │ ✅ free         │ ✅ free              │ ✅ free             │
└──────────────┴─────────────────┴──────────────────────┴─────────────────────┘
```

### Guest "Extract Text" tracking (your question)
Extract Text runs on-device (ML Kit) — **no API cost to you.** So why track it? To create an upgrade funnel: 5 free, then "Register for more." Since it's on-device, the *only* gate is your server counter.

**The right way (since app-local counting is defeated by clearing app data):**
- App calls `POST /v1/guest/extract-text/consume` **before** each on-device OCR run.
- Backend checks `guests/{device_hash}.extract_attempts_used < 5`, increments atomically, returns `{allowed, remaining}`.
- If the app is offline, allow it locally but reconcile on reconnect (Extract Text is free to you, so a few un-counted offline runs cost nothing — be lenient here).

Because it's zero-cost to you, **don't over-engineer guest tracking.** A simple server counter keyed on device hash is enough. The real money protection is on Layout OCR / Scan-to-Word, which require a Firebase token anyway (guests are blocked at the door with a 403).

---

## §4. The credit engine (the core — built correctly)

### 4.1 Unified deduction for BOTH free and pro
Both tiers use the **same** INR-tracking engine. The only difference is where the budget comes from and whether it refills:

| | Free Registered | Pro |
|---|---|---|
| `budget_inr` | 5.00 (one-time) | 50.00 (monthly reset) |
| Refills | Never | On Play renewal (RTDN) |
| On exhaustion | Paywall popup | "Resets on {date}" |

One engine, one code path, configured by tier. This is cleaner than building two systems.

### 4.2 Cost module (single source of pricing)
```python
# app/billing/pricing.py
CREDIT_VALUE_INR = 0.50            # 1 AI credit = ₹0.50
USD_INR_RATE     = 83.5           # config; refresh weekly

SARVAM_LAYOUT_PER_PAGE_INR = 0.50
GEMINI_IN_USD_PER_MTOK     = 0.10
GEMINI_OUT_USD_PER_MTOK    = 0.40

def layout_ocr_inr(pages: int) -> float:
    return round(pages * SARVAM_LAYOUT_PER_PAGE_INR, 4)

def scan_to_word_inr(prompt_tok: int, output_tok: int) -> float:
    usd = (prompt_tok/1e6)*GEMINI_IN_USD_PER_MTOK + (output_tok/1e6)*GEMINI_OUT_USD_PER_MTOK
    return round(usd * USD_INR_RATE, 4)

def inr_to_credits(inr: float) -> float:
    return round(inr / CREDIT_VALUE_INR, 2)     # display only
```

### 4.3 Atomic deduction (Firestore transaction — non-negotiable)
```python
# app/billing/wallet.py
from google.cloud import firestore
db = firestore.AsyncClient()

async def charge(uid: str, cost_inr: float, feature: str, job_id: str) -> dict:
    """Atomically deduct cost from user's budget. Concurrency-safe."""
    ref = db.collection("users").document(uid)

    @firestore.async_transactional
    async def _txn(txn):
        snap = await ref.get(transaction=txn)
        u = snap.to_dict()
        budget = u["ai_budget_inr"]
        spent  = u.get("ai_spent_inr", 0.0)
        if spent + cost_inr > budget:
            return {"allowed": False, "remaining_inr": budget - spent}
        txn.update(ref, {
            "ai_spent_inr": firestore.Increment(cost_inr),
            "last_charge_at": firestore.SERVER_TIMESTAMP,
        })
        return {"allowed": True, "remaining_inr": budget - spent - cost_inr}

    result = await _txn(db.transaction())
    if result["allowed"]:
        await _write_ledger(uid, feature, cost_inr, job_id)   # audit trail
    return result
```

**Why the transaction is mandatory:** double-tap or retry = two concurrent charges. Without the transaction, both read the old balance and both succeed → you overspend. `firestore.Increment` inside `async_transactional` makes it atomic. **Write a test that fires 10 concurrent charges and asserts the balance is correct** — this is the one test you cannot skip.

### 4.4 Request sequencing (Layout OCR / Scan-to-Word)
```
1. get_current_user            → valid Firebase token? (guest → 403 here)
2. tier check                  → free or pro? load budget
3. per-attempt page cap        → Layout ≤15pg, Word ≤100pg, else 400
4. PRE-CHECK                    → would worst-case cost exceed remaining? → 402 paywall
5. call Sarvam / Gemini        → actual work
6. read real cost from response → pages (Sarvam) or usageMetadata (Gemini)
7. charge()                     → atomic deduct + ledger write
8. return result + remaining credits
```
Charge is **post-paid** (you only know real cost after the call), but the **pre-check with a worst-case estimate** bounds any overshoot to a single job. This is the per-attempt cap doing double duty as a safety rail.

---

## §5. Firestore data model

### `users/{uid}`
```json
{
  "uid": "...", "email": "...", "email_verified": true,
  "auth_provider": "google", "tier": "pro", "created_at": "<ts>",

  "ai_budget_inr": 50.0,
  "ai_spent_inr": 6.45,
  "credits_display": 87,

  "subscription": {
    "status": "active",
    "purchase_token": "...",
    "cycle_start": "2026-06-01", "cycle_end": "2026-07-01",
    "auto_renewing": true
  },

  "signup_device_hash": "...", "bound_devices": ["..."]
}
```
> `credits_display` is a **cache**, not truth. Truth = `ai_budget_inr - ai_spent_inr`, converted via `inr_to_credits()`.

### `guests/{device_hash}`
```json
{ "device_hash": "...", "extract_attempts_used": 3, "extract_attempts_max": 5,
  "integrity_verdict": "MEETS_DEVICE_INTEGRITY", "converted_to_uid": null }
```

### `usage_ledger/{auto_id}` — append-only, the real audit trail
```json
{ "uid": "...", "feature": "layout_ocr", "pages": 8,
  "cost_inr": 4.00, "credits": 8.0, "job_id": "...", "created_at": "<ts>" }
```
Source of truth for disputes & margin analysis. Set Firestore **TTL = 90 days** to control cost. Never read on the hot path — write-only.

### `device_registry/{device_hash}` — abuse tracking
```json
{ "device_hash": "...", "associated_uids": ["..."],
  "free_grants_used": 1, "flagged": false, "first_seen": "<ts>" }
```

### `config/pricing` — single editable pricing doc
```json
{ "credit_value_inr": 0.50, "usd_inr_rate": 83.5,
  "sarvam_layout_per_page": 0.50, "max_pages_layout": 15, "max_pages_word": 100 }
```
Change pricing/limits without redeploying. Read once, cache in memory, refresh periodically.

---

## §6. Subscription — Google Play Billing (the only Play-compliant choice)

### Why not Stripe/Razorpay: it's a policy violation, not a preference
Google Play **requires** in-app digital subscriptions to use Play Billing. A ₹99/mo sub unlocking in-app OCR is exactly that. **Stripe/Razorpay here = app rejection/removal.** They're only valid for *website* sales (not your v0). So: **Google Play Billing, settled.**

### Cost-minimal note
Play's fee is 15% (you're under $1M/yr). There's no cheaper *compliant* option for in-app. Don't fight this — it's priced into the ₹19 margin (§0). RevenueCat is **optional** (nice dashboard, free under ~$2.5k/mo tracked revenue) but **not required** — you can verify purchases directly against the Play Developer API for zero added cost. **For v0, skip RevenueCat, verify directly.** Add it later only if managing receipts becomes painful.

### Your three billing endpoints
```
POST /v1/billing/verify    → app sends purchaseToken after purchase;
                             you verify via Play Developer API, flip to Pro,
                             set ai_budget_inr=50, reset ai_spent_inr=0
POST /v1/billing/rtdn      → Pub/Sub push from Play (renew/cancel/grace/refund)
                             drives monthly budget reset — NOT a cron
GET  /v1/billing/status    → app polls entitlement (offline-cacheable)
```

### What credentials you need for billing
1. **Play Developer API access** — a **Google Cloud service account** linked in Play Console (Setup → API access), granted subscription view permissions. Lets your backend call `purchases.subscriptionsv2.get`. **Free.**
2. **Pub/Sub topic** for RTDN — create in Google Cloud, set in Play Console → Monetization setup. Your `/rtdn` endpoint subscribes. **Free at your volume.**
3. The subscription product `pro_99_inr_monthly` created in Play Console.

### Monthly reset is event-driven
When Play sends `SUBSCRIPTION_RENEWED` via RTDN → reset `ai_spent_inr=0`, extend cycle. More reliable than a cron because it fires on the actual payment. Make the handler **idempotent** (Pub/Sub is at-least-once; key on purchaseToken + event ID, ignore replays).

### The "cap hit mid-month" behavior
You **cannot** re-charge an active monthly sub (Play won't allow it). When a Pro user exhausts 100 credits:
- **v0:** show *"You've used this month's AI credits. Resets on {cycle_end}."* Clean, honest, compliant.
- **Post-launch:** sell a **consumable top-up** (separate Play product, e.g. ₹49 → +60 credits). One-time consumable purchase, fully compliant, nice revenue add. Build after v0.

---

## §7. Fraud / abuse prevention (serious — layered, cost-minimal)

Threat: exhaust free credits → clear app data / temp email / reinstall → get another free grant. Layered defense, all free or near-free:

### Layer 1 — Play Integrity API (free, foundational)
Call at `guest/init` and at every registration. Verifies genuine unmodified device + real Play-installed app. Kills emulator farms and tampered APKs. Verify the token **server-side** against Google's API. **Free.**

### Layer 2 — Device binding (defeats app-data wipe)
The free-credit grant is tied to a **device hash** in `device_registry` on *your server*, not app-local state. Clearing app data doesn't reset it.
```python
async def grant_free_credits(uid: str, device_hash: str) -> float:
    reg = await device_registry.get(device_hash)
    if reg and reg.get("free_grants_used", 0) >= 1:
        return 0.0                      # device already used its free grant
    await device_registry.increment(device_hash, "free_grants_used", 1)
    await user_repo.set_budget(uid, 5.0)   # ₹5 = 10 credits
    return 5.0
```

### Layer 3 — Email verification (defeats temp mail, mostly)
- Email/password signups: require Firebase email verification **before** granting credits. Unverified → budget stays ₹0.
- Google sign-in: email already Google-verified → trust it (another reason Google sign-in is the preferred path — lower friction AND harder to abuse).
- Add a **disposable-email domain blocklist** (free open-source lists) — reject temp-mail domains at registration. Low effort, catches lazy abuse.

### Layer 4 — Honest acceptance
A determined abuser with multiple real devices + real Gmails can still get multiple ₹5 grants. **That's fine** — ₹5 is your acquisition cost, and the friction (real device + Play Integrity + verified email) blocks 95%+ of casual abuse. **Don't over-engineer past this for v0.** Your real money is protected by the Pro metered cap, which is airtight (tied to an actual Play payment).

### Tools summary (all free / near-zero)
| Tool | Purpose | Cost |
|---|---|---|
| Play Integrity API | Genuine device/app | Free (quota generous) |
| Device-hash binding | Defeat app-data wipe | Free (Firestore) |
| Firebase email verify | Defeat temp mail | Free (built-in) |
| Disposable-email blocklist | Reject temp domains | Free (OSS list) |
| `usage_ledger` audit | Forensics on abuse | Firestore cost only |

### What to tell the frontend lead
App must: (1) fetch a Play Integrity token at guest-init + registration and send it; (2) generate + persist a stable device ID, send with requests; (3) handle "0 credits granted — device already used free tier" by showing the paywall, not an error.

---

## §8. API surface (all under /v1)

```
# Auth & user
POST   /v1/guest/init                    device + integrity → guest session
POST   /v1/guest/extract-text/consume    decrement 5-attempt counter
POST   /v1/users/sync                     (Bearer) create/refresh user doc
GET    /v1/users/me                       (Bearer) tier + credits (display) + sub

# Usage gating (called by feature-aiocr)
POST   /v1/usage/precheck                 (Bearer) can run feature X? worst-case gate
POST   /v1/usage/commit                   (Bearer) record real cost, atomic deduct
GET    /v1/usage/balance                  (Bearer) remaining credits + %

# Billing
POST   /v1/billing/verify                 (Bearer) verify Play token → Pro
POST   /v1/billing/rtdn                   (Pub/Sub) renew/cancel/grace/refund
GET    /v1/billing/status                 (Bearer) entitlement (cache-friendly)

# Lifecycle (DPDP)
DELETE /v1/users/me                       account deletion (30-day grace)
GET    /v1/users/me/export                data export (email signed link)
```

**Gating pattern:** OCR tool endpoints never touch the wallet directly. They call `usage/precheck` → do work → `usage/commit`. Billing logic stays in ONE place.

---

## §9. Build sequence

**Week 1 — Identity**
1. Generate Firebase Admin key → secret manager → load in FastAPI.
2. `get_current_user` dependency + test with real ID token.
3. `guests/{device_hash}` + `guest/init` with Play Integrity.
4. `guest/extract-text/consume` counter. `GET /users/me`.

**Week 2 — Credit engine**
5. Firestore model: `users`, `guests`, `usage_ledger`, `device_registry`, `config/pricing`.
6. `pricing.py` + `charge()` transaction. **Concurrency test (10 parallel charges).**
7. `usage/precheck` + `usage/commit`. Wire `feature-aiocr` to them.
8. Free-credit grant with device binding.

**Week 3 — Subscription**
9. Play Console: create `pro_99_inr_monthly` + link Cloud service account for Play Developer API + Pub/Sub topic.
10. `billing/verify` against Play Developer API.
11. `billing/rtdn` idempotent handler + monthly budget reset.
12. Grace/expiry → downgrade.

**Week 4 — Hardening + lifecycle**
13. Email-verify gate + disposable-email blocklist.
14. Account deletion + export (DPDP).
15. FX-rate config + weekly refresh.
16. Load-test wallet; validate Gemini real token costs from first 100 jobs; recompute per-page constant.

---

## §10. Gotchas that will bite you

1. **Track INR, display credits.** Never store credits as truth (§0.2). Exchange drift corrupts a credit-based ledger.
2. **Validate Gemini per-page cost with real `usageMetadata`** — image tokens inflate it. The ₹0.03 is an estimate until you measure.
3. **The wallet transaction is mandatory.** Test concurrent charges or you'll leak money.
4. **Round costs UP** to stay margin-safe.
5. **RTDN is at-least-once** — make `/rtdn` idempotent.
6. **Trust Play's `expiryTime`**, never compute cycle end yourself.
7. **Guest counter is server-side** (device-hash keyed), not app-local.
8. **Keep transactions tiny** — no API calls inside the transaction block (Firestore retries on contention).
9. **Restrict the Firebase API key** (image 2) in Cloud Console; **never commit the Admin service-account key.**
10. **GST**: 18% on subs once you cross the registration threshold — talk to a CA before scaling; it's already in the ₹19 margin but verify.

---

## §11. Decisions locked

| Topic | Decision |
|---|---|
| Credit constant | **1 AI credit = ₹0.50** (2 credits = ₹1) |
| Internal unit | **INR, paisa-precision** (credits are display-only) |
| Free Registered | 10 credits (₹5 budget, one-time) |
| Pro | 100 credits (₹50 budget, monthly reset via RTDN) |
| Layout OCR | ₹0.50/pg = 1.0 credit (Sarvam Vision) |
| Scan-to-Word | ~₹0.03/pg = ~0.06 credit (Gemini, **validate real tokens**) |
| Margin | ₹99 → net ~₹69 → worst-case ₹19/user |
| Auth | Frontend owns Google Sign-In ✅ (correct); backend **verifies tokens only** |
| Backend credential | Firebase **Admin** service-account key (you generate) |
| `client_secret.json` | Archived, unused in v0 |
| `google-services.json` | Frontend, not backend |
| Payments | **Google Play Billing** (Stripe = policy violation) |
| RevenueCat | **Skip for v0** (verify Play directly, zero cost) |
| Guest Extract Text | Server counter, device-hash keyed (zero API cost — keep light) |
| Abuse | Play Integrity + device binding + email verify (all free) |
| Source of truth | Append-only `usage_ledger` |

This is production-grade for v0: margin-safe, Play-compliant, abuse-resistant, currency-drift-proof, and honest about where real money protection lives.
