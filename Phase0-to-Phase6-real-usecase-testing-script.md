# Orbit Scanner Backend — Phase 0 to Phase 6 Real Use-Case Testing Script

This document is the complete live/demo testing script for the Orbit Scanner backend from **Phase 0** through **Phase 6**. It is designed for Windows PowerShell and your current backend path:

```powershell
C:\ZPROJECTS\ATOM-PLUTON-TECH-OFFICIAL\Demo-2\Production-code\Official-Backend-Mobile
```

The goal is to prove the backend works from a fresh operational view:

```text
Phase 0 → project structure and base OCR service still runs
Phase 1 → Firebase Admin + Firestore config works
Phase 2 → Firebase Google user auth works
Phase 3 → guest Extract Text counter works
Phase 4 → wallet/credit/ledger engine works
Phase 5 → OCR usage gating + real provider charging works
Phase 6 → admin analytics/export/config controls work
```

> Important: Never paste real API keys, Firebase ID tokens, service account JSON, or admin keys into screenshots, reports, GitHub, or chat. Use placeholders in documentation.

---

## 0. Test Data and Credential Strategy

### Real credentials needed

| Credential / Value | Needed for | Source | Safe handling |
| --- | --- | --- | --- |
| `FIREBASE_CRED_PATH` or `FIREBASE_CRED_JSON` | Firebase Admin + Firestore | Firebase service account | Backend `.env` or Render Secret File only |
| `FIREBASE_PROJECT_ID` | Firebase Admin | Firebase project | `.env` / Render env |
| Fresh Firebase ID token | Phase 2 and Phase 5 user-facing API calls | Android/frontend Firebase Auth | Use in local PowerShell only; expires quickly |
| Optional second Firebase ID token | Wrong-user ownership live test | Another Firebase user | Only needed for final security demo |
| `SARVAM_API_KEY` | Real Layout OCR | Sarvam dashboard | `.env` / Render env only |
| `GEMINI_API_KEY` | Real Scan-to-Word | Google AI Studio | `.env` / Render env only |
| `GEMINI_MODEL` | Gemini model selection | Your `.env` and Firestore config | `gemini-2.5-flash-lite` |
| `ADMIN_API_KEY` | Phase 6 admin endpoints | Generated secret | `.env` / Render env only |
| Normal `X-API-Key` / dev key | Legacy/dev OCR compatibility | Existing backend setting | Not for Android production and not for admin |

### Mock/test values allowed

| Value | Use |
| --- | --- |
| Fake `device_id` GUID | Phase 3 guest counter testing |
| Small one-page PDF/image | Phase 5 real OCR testing |
| Test Firestore budget values | Phase 4/5 wallet testing |
| Test user UID | Scripts/admin user lookup |

### Demo data mode decision

This script supports two demo styles. Choose one before starting.

#### Mode A — Current realistic database mode

Use this if you want to continue from the database you already tested with. Keep these records because Phase 6 analytics can immediately show realistic data:

```text
config/pricing
users/{real Firebase test user}
usage_ledger rows created by real Phase 5 OCR tests
guests test documents from Phase 3
```

This is faster and useful for proving that Phase 6 can read existing production-style data.

#### Mode B — Clean-slate demo mode

Use this if you want to demonstrate Phase 0 through Phase 6 from a fresh operational point. This is the cleaner demo story.

Before starting the live demo, keep only the stable configuration and remove previous test activity:

```text
KEEP:
config/pricing

OPTIONAL KEEP:
users/{real Firebase test user}

DELETE FOR CLEAN-SLATE DEMO:
usage_ledger test rows
guests test documents
old phase4-live-test-* users
old phase4-split-test-* users
old demo/test users you do not need
```

If you keep the real Firebase test user, reset only wallet/testing fields before Phase 5:

```text
ai_budget_inr: 5.0
ai_spent_inr: 0.0
ai_spent_sarvam_inr: 0.0
ai_spent_gemini_inr: 0.0
deleted_at: null
tier: registered
```

If you delete the real Firebase test user, calling `/api/v1/users/me` with a fresh Firebase ID token in Phase 2 should recreate `users/{uid}`. After it is recreated, set its test budget before Phase 5.

In clean-slate mode, the expected collection creation story is:

```text
Start: config/pricing only, plus optionally one reset real test user
Phase 2: /users/me creates or confirms users/{uid}
Phase 3: guest test creates guests/{device_hash}
Phase 4: wallet live script creates temporary users/ledger and then cleans them up
Phase 5: real OCR tests create fresh usage_ledger rows and update users/{uid}
Phase 6: admin analytics/export reads the newly created users, guests, and usage_ledger data
```

For a final delivery demo, Mode B is recommended because it shows every collection being created or updated by the correct phase instead of relying on old test data.

---

## 1. One-Time Local Setup

Open PowerShell.

```powershell
cd "C:\ZPROJECTS\ATOM-PLUTON-TECH-OFFICIAL\Demo-2\Production-code\Official-Backend-Mobile"
.\.venv\Scripts\activate
```

Confirm Python and package environment:

```powershell
python --version
pip --version
```

Confirm `.env` exists:

```powershell
Test-Path .\.env
```

Expected:

```text
True
```

---

## 2. Verify Environment Variables Without Printing Secrets

Run this safe check. It prints only whether secrets exist, not their values.

```powershell
python -c "from dotenv import dotenv_values; e=dotenv_values('.env'); keys=['FIREBASE_PROJECT_ID','FIREBASE_CRED_PATH','FIREBASE_CRED_JSON','SARVAM_API_KEY','GEMINI_API_KEY','GEMINI_MODEL','ADMIN_API_KEY']; [print(k, 'set:', bool(e.get(k)), 'len:', len(e.get(k) or '')) for k in keys]"
```

Expected idea:

```text
FIREBASE_PROJECT_ID set: True
FIREBASE_CRED_PATH set: True
SARVAM_API_KEY set: True
GEMINI_API_KEY set: True
GEMINI_MODEL set: True len: 22
ADMIN_API_KEY set: True len: 64
```

Expected model:

```text
GEMINI_MODEL=gemini-2.5-flash-lite
```

If `ADMIN_API_KEY` is missing, generate it:

```powershell
python -c "import secrets; print(secrets.token_hex(32))"
```

Add it to `.env`:

```env
ADMIN_API_KEY=PASTE_GENERATED_64_CHAR_SECRET_HERE
```

---

## 3. Phase 0 Verification — Base Backend and Existing OCR Structure

### 3.1 Run full tests

```powershell
python -m pytest tests -v
```

Expected after Phase 6 implementation:

```text
130 passed
```

### 3.2 Start server

Open terminal 1:

```powershell
cd "C:\ZPROJECTS\ATOM-PLUTON-TECH-OFFICIAL\Demo-2\Production-code\Official-Backend-Mobile"
.\.venv\Scripts\activate
uvicorn app.main:app --reload --port 8000
```

Keep this terminal running.

Open terminal 2 for API calls:

```powershell
cd "C:\ZPROJECTS\ATOM-PLUTON-TECH-OFFICIAL\Demo-2\Production-code\Official-Backend-Mobile"
.\.venv\Scripts\activate
$BASE="http://localhost:8000/api/v1"
```

### 3.3 Health check

```powershell
curl.exe -s "$BASE/health" | python -m json.tool
```

Expected: healthy response.

### 3.4 Swagger check

Open in browser:

```text
http://localhost:8000/docs
```

Expected:

```text
Swagger opens successfully.
Admin endpoints are not visible.
Normal public/user endpoints are visible.
```

---

## 4. Phase 1 Verification — Firebase Admin + Firestore Pricing Config

### 4.1 Check Firebase Admin connection

If the script exists:

```powershell
python scripts/check_firebase.py
```

Expected:

```text
Firebase Admin OK
Firestore read OK
```

### 4.2 Seed or confirm pricing config

If config is already present, you do not need to reseed. If you need to reseed:

```powershell
python scripts/seed_pricing.py
```

Then verify in Firebase Console:

```text
config/pricing
```

Expected fields:

```text
credit_value_inr: 0.5
free_credits_grant: 10
gemini_in_usd_per_mtok: 0.1
gemini_model: "gemini-2.5-flash-lite"
gemini_out_usd_per_mtok: 0.4
gemini_scan_to_word_conservative_ceiling_inr: 1
guest_extract_attempts_max: 5
max_pages_per_job: 5
sarvam_layout_per_page: 0.5
usd_inr_rate: 95
```

### 4.3 Verify config reload later through Phase 6 admin

This is tested again in Phase 6 using:

```powershell
curl.exe -s -X POST "$BASE/admin/reload-config" -H "X-Admin-API-Key: $ADMIN_API_KEY" | python -m json.tool
```

---

## 5. Phase 2 Verification — Firebase Google User Auth

### 5.1 Get a fresh Firebase ID token

Ask the frontend/Android side to generate a fresh Firebase ID token using Firebase Auth, not the Google ID token.

Android/Kotlin source:

```kotlin
FirebaseAuth.getInstance().currentUser?.getIdToken(true)
```

Set it locally:

```powershell
$TOKEN="PASTE_FRESH_FIREBASE_ID_TOKEN_HERE"
```

Do not print the token.

### 5.2 Call `/users/me`

```powershell
$me = Invoke-RestMethod -Uri "$BASE/users/me" -Headers @{ Authorization = "Bearer $TOKEN" }
$me
$UID=$me.uid
$UID
```

Expected:

```text
uid exists
email exists
tier = registered
ai_budget_inr / ai_spent_inr fields exist
```

Firestore should contain:

```text
users/{uid}
```

Expected user fields:

```text
uid
email
tier: registered
ai_budget_inr
ai_spent_inr
ai_spent_sarvam_inr
ai_spent_gemini_inr
lifetime_sarvam_inr
lifetime_gemini_inr
created_at
deleted_at: null
```

### 5.3 Negative auth test

```powershell
curl.exe -i "$BASE/users/me"
```

Expected:

```text
401 Unauthorized
```

---

## 6. Phase 3 Verification — Guest Extract Text Attempt Counter

Phase 3 tracks guest usage for on-device Extract Text. The backend does not receive OCR image/text. It only counts attempts.

### 6.1 Create fake stable device ID

```powershell
$DEVICE_ID=[guid]::NewGuid().ToString()
$DEVICE_ID
```

### 6.2 Guest init

```powershell
$body=@{ device_id=$DEVICE_ID } | ConvertTo-Json -Compress
Invoke-RestMethod -Uri "$BASE/guest/init" -Method POST -ContentType "application/json" -Body $body
```

Expected:

```text
remaining = 5
max_attempts = 5
```

### 6.3 Consume guest attempts

Run this loop:

```powershell
1..6 | ForEach-Object {
  $body=@{ device_id=$DEVICE_ID } | ConvertTo-Json -Compress
  Invoke-RestMethod -Uri "$BASE/guest/extract-text/consume" -Method POST -ContentType "application/json" -Body $body
}
```

Expected pattern:

```text
Attempt 1 → allowed true, remaining 4
Attempt 2 → allowed true, remaining 3
Attempt 3 → allowed true, remaining 2
Attempt 4 → allowed true, remaining 1
Attempt 5 → allowed true, remaining 0
Attempt 6 → allowed false, remaining 0
```

Firestore should contain:

```text
guests/{device_hash}
extract_attempts_used: 5
extract_attempts_max: 5
```

---

## 7. Phase 4 Verification — Credit Engine, Wallet, and Ledger

Phase 4 proves atomic wallet charging and ledger creation.

### 7.1 Run wallet unit tests

```powershell
python -m pytest tests/account/test_wallet.py -v
```

Expected:

```text
passed
```

### 7.2 Run live wallet script if available

Use cleanup mode by default so temporary Phase 4 data does not remain.

```powershell
python scripts/check_wallet_live.py
```

Expected:

```text
RESULT: ALL CHECKS PASSED
```

This proves:

```text
concurrent charges are safe
overspend is rejected
provider split works
ledger is created atomically
wallet totals match ledger rows
```

### 7.3 Verify no old Phase 4 test clutter remains, if cleanup script exists

Dry-run/default preview:

```powershell
python scripts/cleanup_phase4_test_data.py
```

If it identifies only `phase4-live-test-*` or `phase4-split-test-*` records and you want to clean them:

```powershell
python scripts/cleanup_phase4_test_data.py --confirm
```

Do not delete Phase 5 real usage rows.

---

## 8. Phase 5 Verification — Usage Gating + Existing OCR Integration

Phase 5 is the most important production flow. It proves user-authenticated OCR usage is charged internally after real provider usage.

### 8.1 Prepare test user budget

In Firebase Console, open:

```text
users/{your Firebase test uid}
```

Set for clean testing:

```text
ai_budget_inr: 5.0
ai_spent_inr: 0.0
ai_spent_sarvam_inr: 0.0
ai_spent_gemini_inr: 0.0
```

You may keep lifetime fields as-is. For a disposable test account, you can reset lifetime fields too.

### 8.2 Check balance

```powershell
Invoke-RestMethod -Uri "$BASE/usage/balance" -Headers @{ Authorization = "Bearer $TOKEN" }
```

Expected:

```text
ai_budget_inr = 5.0
ai_spent_inr = 0.0
remaining_inr = 5.0
credits are derived from INR
```

### 8.3 Page cap precheck — 6 pages should return 422

```powershell
$body=@{ feature="layout_ocr"; pages=6 } | ConvertTo-Json -Compress
try {
  Invoke-RestMethod -Uri "$BASE/usage/precheck" -Method POST -Headers @{ Authorization="Bearer $TOKEN"; "Content-Type"="application/json" } -Body $body
} catch {
  $_.Exception.Response.StatusCode.value__
}
```

Expected:

```text
422
```

### 8.4 Over-budget precheck — should return 402

Temporarily set user in Firestore:

```text
ai_budget_inr: 0.0
ai_spent_inr: 0.0
```

Then run:

```powershell
$body=@{ feature="scan_to_word"; pages=1 } | ConvertTo-Json -Compress
try {
  Invoke-RestMethod -Uri "$BASE/usage/precheck" -Method POST -Headers @{ Authorization="Bearer $TOKEN"; "Content-Type"="application/json" } -Body $body
} catch {
  $_.Exception.Response.StatusCode.value__
}
```

Expected:

```text
402
```

This should create:

```text
no job
no provider call
no usage_ledger row
no wallet charge
```

Restore budget:

```text
ai_budget_inr: 5.0
ai_spent_inr: 0.0
ai_spent_sarvam_inr: 0.0
ai_spent_gemini_inr: 0.0
```

### 8.5 Prepare a small real OCR test file

Use a small, clear, one-page PDF/image.

```powershell
$FILE="C:\path\to\small-one-page-test.pdf"
Test-Path $FILE
```

Expected:

```text
True
```

### 8.6 Layout OCR real provider flow

Check balance before:

```powershell
$balanceBeforeLayout = Invoke-RestMethod -Uri "$BASE/usage/balance" -Headers @{ Authorization="Bearer $TOKEN" }
$balanceBeforeLayout
```

Submit Layout OCR:

```powershell
$layoutJob = curl.exe -s -X POST "$BASE/ocr/layout-extract" `
  -H "Authorization: Bearer $TOKEN" `
  -F "file=@$FILE" `
  -F "output_format=docx" | ConvertFrom-Json

$layoutJob
$LAYOUT_JOB_ID=$layoutJob.job_id
$LAYOUT_JOB_ID
```

Poll status:

```powershell
Invoke-RestMethod -Uri "$BASE/jobs/$LAYOUT_JOB_ID/status" -Headers @{ Authorization="Bearer $TOKEN" }
```

Repeat until completed.

Download result using GET:

```powershell
curl.exe -L "$BASE/jobs/$LAYOUT_JOB_ID/download" `
  -H "Authorization: Bearer $TOKEN" `
  -o "layout_result.docx"

Get-Item .\layout_result.docx
```

Expected:

```text
file exists and size > 0
```

Check balance after:

```powershell
$balanceAfterLayout = Invoke-RestMethod -Uri "$BASE/usage/balance" -Headers @{ Authorization="Bearer $TOKEN" }
$balanceAfterLayout
```

Expected for one page:

```text
ai_spent_inr increases by about 0.5
ai_spent_sarvam_inr increases by about 0.5
remaining_inr decreases by about 0.5
```

Firestore `usage_ledger` expected row:

```text
uid: your UID
job_id: $LAYOUT_JOB_ID
feature: layout_ocr
provider: sarvam
model_id: sarvam-layout-ocr
pages: 1
cost_inr: 0.5
credits: 1
pricing_currency: INR
usd_inr_rate_used: null
```

### 8.7 Scan-to-Word real provider flow

Submit Scan-to-Word:

```powershell
$scanJob = curl.exe -s -X POST "$BASE/ocr/scan-to-word" `
  -H "Authorization: Bearer $TOKEN" `
  -F "file=@$FILE" | ConvertFrom-Json

$scanJob
$SCAN_JOB_ID=$scanJob.job_id
$SCAN_JOB_ID
```

Poll status:

```powershell
Invoke-RestMethod -Uri "$BASE/jobs/$SCAN_JOB_ID/status" -Headers @{ Authorization="Bearer $TOKEN" }
```

Repeat until completed.

Download result:

```powershell
curl.exe -L "$BASE/jobs/$SCAN_JOB_ID/download" `
  -H "Authorization: Bearer $TOKEN" `
  -o "scan_to_word_result.docx"

Get-Item .\scan_to_word_result.docx
```

Check balance after:

```powershell
$balanceAfterScan = Invoke-RestMethod -Uri "$BASE/usage/balance" -Headers @{ Authorization="Bearer $TOKEN" }
$balanceAfterScan
```

Expected:

```text
ai_spent_gemini_inr increases by real token-calculated cost
ai_spent_inr increases
remaining_inr decreases
```

Firestore `usage_ledger` expected row:

```text
uid: your UID
job_id: $SCAN_JOB_ID
feature: scan_to_word
provider: gemini
model_id: gemini-2.5-flash-lite
prompt_tokens: present
output_tokens: present
thoughts_tokens: present or 0
pricing_currency: USD
usd_inr_rate_used: 95
cost_inr: calculated from tokens
```

### 8.8 Missing token test

```powershell
curl.exe -i -X POST "$BASE/ocr/layout-extract" `
  -F "file=@$FILE" `
  -F "output_format=docx"
```

Expected:

```text
401 Unauthorized
```

### 8.9 Wrong-user ownership blocking

This requires a second Firebase user token.

```powershell
$OTHER_TOKEN="PASTE_SECOND_USER_FIREBASE_ID_TOKEN_HERE"
```

Test status:

```powershell
curl.exe -i "$BASE/jobs/$LAYOUT_JOB_ID/status" -H "Authorization: Bearer $OTHER_TOKEN"
```

Test download:

```powershell
curl.exe -i "$BASE/jobs/$LAYOUT_JOB_ID/download" -H "Authorization: Bearer $OTHER_TOKEN"
```

Expected:

```text
403 Forbidden or 404 Not Found
```

If you do not have a second Firebase user during the demo, mark this as:

```text
Pending live test, but covered by automated tests in tests/ai_ocr/test_jobs.py
```

### 8.10 Phase 5 focused tests

```powershell
python -m pytest tests/account/test_usage.py -v
python -m pytest tests/ai_ocr/test_jobs.py -v
python -m pytest tests/ai_ocr -v
```

Expected:

```text
passed
```

---

## 9. Phase 6 Verification — Real-Time Analytics and Admin

Phase 6 proves admin-only analytics/export/config endpoints work and are protected by `X-Admin-API-Key`.

### 9.1 Set admin key variable

Set it from your `.env` value manually. Do not print it in reports.

```powershell
$ADMIN_API_KEY="PASTE_ADMIN_API_KEY_FROM_ENV_HERE"
```

Safe key existence check:

```powershell
python -c "from dotenv import dotenv_values; e=dotenv_values('.env'); v=e.get('ADMIN_API_KEY') or ''; print('ADMIN_API_KEY set:', bool(v)); print('length:', len(v))"
```

Expected:

```text
ADMIN_API_KEY set: True
length: 64
```

### 9.2 Admin analytics summary

```powershell
curl.exe -s "$BASE/admin/analytics/summary" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" | python -m json.tool
```

Expected fields:

```text
total_users
active_users
deleted_users
total_guests
total_ai_budget_inr
total_ai_spent_inr
total_ai_spent_sarvam_inr
total_ai_spent_gemini_inr
total_remaining_inr
pricing_credit_value_inr
pricing_max_pages_per_job
pricing_gemini_model
pricing_usd_inr_rate
```

This proves current Firestore rollups are readable in real time.

### 9.3 CSV export — active users only

```powershell
curl.exe -L "$BASE/admin/usage-export" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" `
  -o usage_export.csv

Get-Item .\usage_export.csv
Get-Content .\usage_export.csv -TotalCount 5
```

Expected headers:

```text
email,uid,tier,created_at,deleted_at,ai_budget_inr,ai_spent_sarvam_inr,ai_spent_gemini_inr,ai_spent_inr,remaining_inr,credits_remaining,lifetime_sarvam_inr,lifetime_gemini_inr,subscription_status
```

### 9.4 CSV export — include deleted users

```powershell
curl.exe -L "$BASE/admin/usage-export?include_deleted=true" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" `
  -o usage_export_all.csv

Get-Item .\usage_export_all.csv
Get-Content .\usage_export_all.csv -TotalCount 5
```

Expected: file downloads successfully.

### 9.5 Reconcile CSV totals with analytics summary

```powershell
$csv = Import-Csv .\usage_export.csv
$totalSpentFromCsv = ($csv | ForEach-Object { [double]$_.ai_spent_inr } | Measure-Object -Sum).Sum
$totalSarvamFromCsv = ($csv | ForEach-Object { [double]$_.ai_spent_sarvam_inr } | Measure-Object -Sum).Sum
$totalGeminiFromCsv = ($csv | ForEach-Object { [double]$_.ai_spent_gemini_inr } | Measure-Object -Sum).Sum

"CSV total ai_spent_inr: $totalSpentFromCsv"
"CSV total sarvam: $totalSarvamFromCsv"
"CSV total gemini: $totalGeminiFromCsv"

curl.exe -s "$BASE/admin/analytics/summary" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" `
  -o admin_summary.json

Get-Content .\admin_summary.json | python -m json.tool
```

Expected:

```text
CSV total ai_spent_inr roughly equals summary.total_ai_spent_inr
CSV total sarvam roughly equals summary.total_ai_spent_sarvam_inr
CSV total gemini roughly equals summary.total_ai_spent_gemini_inr
```

Small float formatting differences are acceptable.

### 9.6 Reload config endpoint

```powershell
curl.exe -s -X POST "$BASE/admin/reload-config" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" | python -m json.tool
```

Expected response includes current pricing config:

```text
credit_value_inr
gemini_model
max_pages_per_job
sarvam_layout_per_page
usd_inr_rate
```

Optional stronger test:

```text
1. Temporarily change config/pricing.usd_inr_rate in Firestore from 95 to 96.
2. Call /admin/reload-config.
3. Confirm response shows 96.
4. Change it back to 95.
5. Call /admin/reload-config again.
```

### 9.7 Admin user detail

```powershell
$TEST_UID=$UID
curl.exe -s "$BASE/admin/users/$TEST_UID" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" | python -m json.tool
```

Expected:

```text
uid
email
tier
ai_budget_inr
ai_spent_inr
ai_spent_sarvam_inr
ai_spent_gemini_inr
remaining_inr
credits_remaining
```

### 9.8 Wrong admin key must fail

```powershell
curl.exe -i "$BASE/admin/analytics/summary" `
  -H "X-Admin-API-Key: wrong"
```

Expected:

```text
401 Unauthorized
```

### 9.9 Missing admin key must fail

```powershell
curl.exe -i "$BASE/admin/analytics/summary"
```

Expected:

```text
401 Unauthorized
```

### 9.10 Normal `X-API-Key` must not authorize admin

```powershell
$NORMAL_API_KEY="PASTE_NORMAL_X_API_KEY_IF_AVAILABLE"

curl.exe -i "$BASE/admin/analytics/summary" `
  -H "X-API-Key: $NORMAL_API_KEY"
```

Expected:

```text
401 Unauthorized
```

### 9.11 Firebase token must not authorize admin

```powershell
curl.exe -i "$BASE/admin/analytics/summary" `
  -H "Authorization: Bearer $TOKEN"
```

Expected:

```text
401 Unauthorized
```

### 9.12 Admin endpoints hidden from Swagger

```powershell
curl.exe -s "http://localhost:8000/openapi.json" | python -c "import sys,json; d=json.load(sys.stdin); admin=[p for p in d['paths'] if '/admin/' in p]; print('Admin paths in Swagger:', admin)"
```

Expected:

```text
Admin paths in Swagger: []
```

---

## 10. Final End-to-End Pass Criteria

The demo passes when all are true:

```text
✅ Full automated test suite passes: 130/130
✅ Health endpoint works
✅ Swagger opens and admin endpoints are hidden
✅ Firestore config/pricing exists and has correct model/pricing values
✅ Fresh Firebase ID token works with /users/me
✅ Guest init and consume counter works
✅ Wallet live script passes or wallet tests pass
✅ /usage/precheck returns 422 for 6 pages when cap is 5
✅ /usage/precheck returns 402 for no budget
✅ Layout OCR creates uid-owned job
✅ Layout OCR charges Sarvam usage after completion
✅ Layout OCR creates usage_ledger row with feature/provider/model_id/pages
✅ Scan-to-Word creates uid-owned job
✅ Scan-to-Word charges Gemini token usage after completion
✅ Scan-to-Word creates usage_ledger row with tokens/model_id/usd_inr_rate_used
✅ /usage/balance updates after charges
✅ Own user can download completed job
✅ Wrong-user live test passes or is recorded as pending with automated coverage
✅ Admin analytics summary works with X-Admin-API-Key
✅ Admin CSV export works and totals reconcile
✅ Admin reload-config works
✅ Wrong/missing admin key fails
✅ Firebase token and normal X-API-Key cannot access admin
```

---

## 11. Final Secret and Git Safety Check Before Commit/Push

Run tests one last time:

```powershell
python -m pytest tests -v
```

Check for secrets before staging:

```powershell
git status --short --untracked-files=all | Select-String -Pattern "\.env$|firebase-adminsdk|serviceAccount|firebase-admin\.json|SECRETS|SARVAM|GEMINI|ADMIN_API_KEY|API_KEY"
```

Expected: no secret files/values staged or untracked for commit.

Stage and check again:

```powershell
git add -A

git diff --cached --name-only | Select-String -Pattern "\.env$|firebase-adminsdk|serviceAccount|firebase-admin\.json|SECRETS"
```

Expected: no output.

Commit examples:

```powershell
git commit -m "feat(auth): phase 6 add admin analytics and export"
git push
```

If this document itself is committed, make sure it contains only placeholders, not real secrets.

---

## 12. Demo Report Template

Use this at the end of the live demo:

```text
Orbit Scanner Backend Phase 0–6 Demo Result

Automated tests: 130/130 passed
Health: passed
Firebase Admin/Firestore: passed
Pricing config: passed
Firebase user auth: passed
Guest counter: passed
Wallet/ledger: passed
Layout OCR real provider: passed
Scan-to-Word real provider: passed
Usage gating 422/402: passed
Job ownership: passed / pending live second-user test
Admin summary: passed
CSV export: passed
Config reload: passed
Admin security: passed
Swagger hidden admin paths: passed

Notes:
- Android production flow uses Firebase Bearer token.
- Credit deduction happens inside backend worker after real provider usage.
- usage_ledger stores feature/provider/model_id/job_id/uid.
- Admin endpoints use X-Admin-API-Key and are hidden from Swagger.
```
