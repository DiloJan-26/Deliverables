## How to check token type yourself without leaking it

In PowerShell, after setting:

```powershell
$TOKEN="paste-token-here"
$env:TOKEN=$TOKEN
```

Run:

```powershell
@'
import os, json, base64, datetime

token = os.environ.get("TOKEN", "")
parts = token.split(".")

print("JWT parts:", len(parts))

if len(parts) == 3:
    payload_b64 = parts[1] + "=" * (-len(parts[1]) % 4)
    payload = json.loads(base64.urlsafe_b64decode(payload_b64.encode()))

    print("iss:", payload.get("iss"))
    print("aud:", payload.get("aud"))
    print("has firebase claim:", "firebase" in payload)
    print("sign_in_provider:", payload.get("firebase", {}).get("sign_in_provider"))
    print("uid/user_id:", payload.get("user_id") or payload.get("sub"))

    exp = payload.get("exp")
    if exp:
        print("expires_at:", datetime.datetime.fromtimestamp(exp).isoformat())
else:
    print("Not a normal JWT")
'@ | python -
```

If it prints:

```text
iss: https://accounts.google.com
has firebase claim: False
```

then it is the **wrong token**.

Correct token should print something like:

```text
iss: https://securetoken.google.com/orbit-scanner
aud: orbit-scanner
has firebase claim: True
sign_in_provider: google.com
```


For your backend, the correct Firebase token should have issuer like:

```text
iss = https://securetoken.google.com/orbit-scanner
aud = orbit-scanner
firebase.sign_in_provider = google.com
```

Firebase Admin SDK verifies **Firebase ID tokens**, and Firebase’s Android SDK provides that token using `FirebaseUser.getIdToken(forceRefresh)` for backend authentication. ([Firebase][1]) Firebase’s decoded token docs also show the expected issuer format as `https://securetoken.google.com/<PROJECT_ID>`. ([Firebase][2])

So your backend error is correct:

```json
{"error":"invalid_token","message":"Firebase ID token is invalid or malformed.","code":401}
```

## Problem is from frontend token extraction

They are probably giving you this not the right token:

```text
GoogleSignInAccount.idToken
or Credential Manager googleIdTokenCredential.idToken
```

But they must give you this:

```kotlin
FirebaseAuth.getInstance().currentUser?.getIdToken(true)
```

Correct Android flow should be:

```kotlin
val credential = GoogleAuthProvider.getCredential(googleIdToken, null)

FirebaseAuth.getInstance()
    .signInWithCredential(credential)
    .addOnSuccessListener {
        FirebaseAuth.getInstance().currentUser?.getIdToken(true)
            ?.addOnSuccessListener { result ->
                val firebaseIdToken = result.token
                Log.d("FIREBASE_ID_TOKEN", firebaseIdToken ?: "NO_TOKEN")
            }
    }
```

The important difference:

```text
googleIdToken = used to sign into Firebase
firebaseIdToken = sent to your FastAPI backend
```



## Final verdict

Your backend is most likely fine.

Ask frontend this exact thing:

```text
Please send the Firebase ID token from FirebaseAuth.getInstance().currentUser?.getIdToken(true), not the Google ID token from Google Sign-In or Credential Manager.
```

[1]: https://firebase.google.com/docs/auth/admin/verify-id-tokens?utm_source=chatgpt.com "Verify ID Tokens | Firebase Authentication - Google"
[2]: https://firebase.google.com/docs/reference/admin/node/firebase-admin.auth.decodedidtoken?utm_source=chatgpt.com "DecodedIdToken interface | Firebase Admin SDK - Google"
