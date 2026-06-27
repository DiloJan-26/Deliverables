Yes. Since `130/130 passed`, now do **live Phase 6 admin verification** against your real Firestore data.

Do **not paste your `ADMIN_API_KEY` here**. Keep it local only.

---

# Phase 6 Live Testing Procedure

## 1. Run full automated tests first

```powershell id="oihz8q"
cd "C:\ZPROJECTS\ATOM-PLUTON-TECH-OFFICIAL\Demo-2\Production-code\Official-Backend-Mobile"
.\.venv\Scripts\activate

python -m pytest tests -v
```

Expected:

```text id="g2kkko"
130 passed
```

---

## 2. Verify `ADMIN_API_KEY` exists

```powershell id="zdodir"
python -c "from dotenv import dotenv_values; e=dotenv_values('.env'); v=e.get('ADMIN_API_KEY') or ''; print('ADMIN_API_KEY set:', bool(v)); print('length:', len(v))"
```

Expected:

```text id="e0ckd9"
ADMIN_API_KEY set: True
length: 64
```

---

## 3. Start backend server

Open PowerShell terminal 1:

```powershell id="m399kk"
cd "C:\ZPROJECTS\ATOM-PLUTON-TECH-OFFICIAL\Demo-2\Production-code\Official-Backend-Mobile"
.\.venv\Scripts\activate

uvicorn app.main:app --reload --port 8000
```

Keep it running.

---

## 4. Set test variables

Open PowerShell terminal 2:

```powershell id="l7phwz"
cd "C:\ZPROJECTS\ATOM-PLUTON-TECH-OFFICIAL\Demo-2\Production-code\Official-Backend-Mobile"
.\.venv\Scripts\activate

$BASE = "http://localhost:8000/api/v1"
$ADMIN_API_KEY = "PASTE_YOUR_ADMIN_API_KEY_FROM_DOTENV_HERE"
```

Do not print the key after setting it.

---

# 5. Test admin summary endpoint

```powershell id="jguk5h"
curl.exe -s "$BASE/admin/analytics/summary" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" | python -m json.tool
```

Expected: JSON summary with fields like:

```text id="vw8a1v"
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

This proves:

```text id="j9f9yt"
✅ admin endpoint works
✅ ADMIN_API_KEY auth works
✅ summary reads current Firestore data
✅ Phase 4/5 rollups are usable for analytics
```

---

# 6. Test CSV export

```powershell id="ofcjjh"
curl.exe -L "$BASE/admin/usage-export" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" `
  -o usage_export.csv
```

Check file exists:

```powershell id="2ckzat"
Get-Item .\usage_export.csv
```

Preview headers and first rows:

```powershell id="guy403"
Get-Content .\usage_export.csv -TotalCount 5
```

Expected headers should include:

```text id="erdk9p"
email
uid
tier
created_at
deleted_at
ai_budget_inr
ai_spent_sarvam_inr
ai_spent_gemini_inr
ai_spent_inr
remaining_inr
credits_remaining
lifetime_sarvam_inr
lifetime_gemini_inr
subscription_status
```

This proves:

```text id="aejrec"
✅ CSV export works
✅ user rollup fields export correctly
✅ no usage_ledger scan is needed for normal export
```

---

# 7. Test CSV export with deleted users included

```powershell id="edq9e9"
curl.exe -L "$BASE/admin/usage-export?include_deleted=true" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" `
  -o usage_export_all.csv
```

Check:

```powershell id="8yqz3w"
Get-Item .\usage_export_all.csv
Get-Content .\usage_export_all.csv -TotalCount 5
```

Expected: file downloads successfully. If you do not have deleted users, it may look similar to the normal export.

---

# 8. Reconcile CSV numbers with summary

Load CSV:

```powershell id="2qx636"
$csv = Import-Csv .\usage_export.csv

$csv.Count
$csv | Select-Object -First 3
```

Calculate total spend from CSV:

```powershell id="owr51m"
$totalSpentFromCsv = ($csv | ForEach-Object { [double]$_.ai_spent_inr } | Measure-Object -Sum).Sum
$totalSarvamFromCsv = ($csv | ForEach-Object { [double]$_.ai_spent_sarvam_inr } | Measure-Object -Sum).Sum
$totalGeminiFromCsv = ($csv | ForEach-Object { [double]$_.ai_spent_gemini_inr } | Measure-Object -Sum).Sum

"CSV total ai_spent_inr: $totalSpentFromCsv"
"CSV total sarvam: $totalSarvamFromCsv"
"CSV total gemini: $totalGeminiFromCsv"
```

Then get summary into a file:

```powershell id="xucslz"
curl.exe -s "$BASE/admin/analytics/summary" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" `
  -o admin_summary.json

Get-Content .\admin_summary.json | python -m json.tool
```

Compare:

```text id="stzrus"
CSV total ai_spent_inr ≈ summary.total_ai_spent_inr
CSV total sarvam ≈ summary.total_ai_spent_sarvam_inr
CSV total gemini ≈ summary.total_ai_spent_gemini_inr
```

Small float formatting differences are okay.

This proves:

```text id="oix8r5"
✅ CSV numbers reconcile with user docs
✅ summary totals match exported user rollups
```

---

# 9. Test reload config endpoint

```powershell id="ypxckk"
curl.exe -s -X POST "$BASE/admin/reload-config" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" | python -m json.tool
```

Expected response includes current pricing config:

```text id="rshkyd"
credit_value_inr
gemini_model
gemini_in_usd_per_mtok
gemini_out_usd_per_mtok
max_pages_per_job
sarvam_layout_per_page
usd_inr_rate
```

This proves:

```text id="8kpvx5"
✅ reload-config endpoint works
✅ endpoint uses existing pricing reload function
✅ admin can refresh pricing without backend restart
```

Optional stronger test:

```text id="6o5k8l"
1. Temporarily change config/pricing.usd_inr_rate in Firestore from 95 to 96.
2. Call /admin/reload-config.
3. Confirm response shows 96.
4. Change it back to 95.
5. Call /admin/reload-config again.
```

Do this only if you are comfortable editing Firestore config.

---

# 10. Test admin user detail endpoint

Use your real test UID from Firestore.

```powershell id="dnhffy"
$TEST_UID = "PASTE_REAL_TEST_USER_UID_HERE"

curl.exe -s "$BASE/admin/users/$TEST_UID" `
  -H "X-Admin-API-Key: $ADMIN_API_KEY" | python -m json.tool
```

Expected fields:

```text id="r98my9"
uid
email
tier
ai_budget_inr
ai_spent_inr
ai_spent_sarvam_inr
ai_spent_gemini_inr
remaining_inr
credits_remaining
lifetime_sarvam_inr
lifetime_gemini_inr
```

This proves:

```text id="f3chf2"
✅ admin can inspect one user safely
✅ wallet rollups are visible for admin
✅ no provider secrets are returned
```

---

# 11. Test wrong admin key

```powershell id="mesjfg"
curl.exe -i "$BASE/admin/analytics/summary" `
  -H "X-Admin-API-Key: wrong"
```

Expected:

```text id="hxdu57"
401 Unauthorized
```

This proves:

```text id="4j0pvu"
✅ wrong admin key is blocked
```

---

# 12. Test missing admin key

```powershell id="wiy6qd"
curl.exe -i "$BASE/admin/analytics/summary"
```

Expected:

```text id="k7tzth"
401 Unauthorized
```

This proves:

```text id="m5sklh"
✅ missing admin key is blocked
```

---

# 13. Test normal `X-API-Key` does not authorize admin

Use your normal internal API key if you have it.

```powershell id="zzfeyv"
$NORMAL_API_KEY = "PASTE_NORMAL_X_API_KEY_HERE"

curl.exe -i "$BASE/admin/analytics/summary" `
  -H "X-API-Key: $NORMAL_API_KEY"
```

Expected:

```text id="4y0dq9"
401 Unauthorized
```

This proves:

```text id="mdrbto"
✅ normal OCR/dev X-API-Key cannot access admin endpoints
```

---

# 14. Test Firebase token does not authorize admin

Use any Firebase ID token if you still have one.

```powershell id="w8ga12"
$TOKEN = "PASTE_FIREBASE_ID_TOKEN_HERE"

curl.exe -i "$BASE/admin/analytics/summary" `
  -H "Authorization: Bearer $TOKEN"
```

Expected:

```text id="7id020"
401 Unauthorized
```

This proves:

```text id="ublf87"
✅ Firebase user token cannot access admin endpoints
✅ admin access is separated from normal user access
```

---

# 15. Test admin endpoints are hidden from Swagger/OpenAPI

```powershell id="snra01"
curl.exe -s "http://localhost:8000/openapi.json" | python -c "import sys,json; d=json.load(sys.stdin); admin=[p for p in d['paths'] if '/admin/' in p]; print('Admin paths in Swagger:', admin)"
```

Expected:

```text id="vmnhu4"
Admin paths in Swagger: []
```

This proves:

```text id="eap87a"
✅ admin endpoints are hidden from public Swagger docs
```

---

# 16. Confirm old app behavior still works

Run focused tests:

```powershell id="tenw26"
python -m pytest tests/account/test_admin.py -v
python -m pytest tests/account/test_usage.py -v
python -m pytest tests/ai_ocr/test_jobs.py -v
python -m pytest tests/ai_ocr -v
```

Then full suite:

```powershell id="g5bswb"
python -m pytest tests -v
```

Expected:

```text id="6r68r6"
130 passed
```

---

# Final Phase 6 checklist

Mark Phase 6 verified when:

```text id="a1zump"
✅ ADMIN_API_KEY exists and length is 64
✅ 130 automated tests pass
✅ /admin/analytics/summary works with correct admin key
✅ /admin/usage-export downloads CSV
✅ CSV headers are correct
✅ CSV totals reconcile with summary/user rollups
✅ /admin/usage-export?include_deleted=true works
✅ /admin/reload-config works
✅ /admin/users/{uid} works
✅ wrong admin key returns 401
✅ missing admin key returns 401
✅ normal X-API-Key cannot access admin
✅ Firebase Bearer token cannot access admin
✅ admin endpoints are hidden from Swagger
```
