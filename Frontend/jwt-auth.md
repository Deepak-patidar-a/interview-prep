# JWT Auth + Refresh Tokens

## Question
Walk me through the complete JWT authentication flow — from login to token expiry. Where do you store tokens, how does the refresh flow work, and what happens when the refresh token expires?

---

## My Answer
When a user logs in, we call the auth API which checks if the user exists. If new, we register them. On success, we get a JWT token which we store in a cookie and also in React Context for protected route checks. We have an access token and a refresh token. When the refresh token expires, we generate a new token. Tokens are stored in cookies on the frontend.

## What I Got Right
- Correct login flow — check user → register if new → issue tokens
- Using Context for protected route checks
- Storing tokens in cookies (better than localStorage)
- Knowing refresh token generates a new access token

## What I Got Wrong
- I had the expiry timings backwards. Access token should be short-lived, refresh token should be long-lived. Not the other way around.

---

## Model Answer
When a user logs in, the server returns two tokens — a short-lived access token (15 minutes) and a long-lived refresh token (7 days), both stored in HttpOnly cookies so JavaScript cannot access them, protecting against XSS. On every API call, we send the access token. When it expires, our Axios interceptor catches the 401 response, automatically calls the refresh endpoint with the refresh token, gets a new access token, and retries the original request silently. The user never sees a logout. If the refresh token itself expires, we redirect the user to the login page — there is no way around re-authentication at that point.

---

## Key Things to Remember

**Token expiry rule:**
- Access token = short (15 min to 1 hour) — travels with every request, higher risk if stolen
- Refresh token = long (7 days to 30 days) — sits in cookie, used occasionally, lower exposure

**Memory trick:** Access token = hotel key card (use constantly, deactivates quickly). Refresh token = passport (keep it safe, valid for a long time).

**Cookie flags that matter:**
- `HttpOnly` — JavaScript cannot read the cookie, protects against XSS
- `Secure` — cookie only travels over HTTPS
- `SameSite=Strict` — protects against CSRF

**JWT is encoded, not encrypted:**
- Anyone can decode the payload with base64 decode
- Only the signature is hashed with a secret key to prevent tampering
- Never store sensitive data like passwords inside JWT payload

**Axios interceptor flow:**
1. API call fails with 401
2. Interceptor catches it
3. Calls refresh endpoint silently
4. Gets new access token
5. Retries original request
6. User never notices

---

## Things to Improve
- Learn how to write an Axios interceptor for refresh token flow
- Confirm exact expiry timings used in the project with the team
