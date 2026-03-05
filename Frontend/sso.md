# SSO — Single Sign On

## Question
How does SSO work in your application? What happens when a user clicks Login with SSO? Who is the Identity Provider? How does your frontend and backend handle the SSO response? How does SSO coexist with your existing JWT auth?

---

## My Answer
SSO at enterprise level means if you have company system login credentials, you can use those to log into our website instead of a separate password. It is high level security because the website opens only with company authorized credentials. Our Identity Provider is the company's database system. We are not running SSO and JWT in parallel — we are moving some customers to SSO and keeping others on JWT.

## What I Got Right
- Correct business understanding of SSO
- Correctly identified that SSO and JWT coexist for different customers during migration
- Understood SSO is more secure because company controls access

## What I Got Wrong
- Identity Provider is not the company database directly. The IDP is a dedicated auth service like Azure AD, Okta, or ADFS that sits in front of the database.

---

## Model Answer
We recently started implementing SSO for enterprise customers using OIDC protocol with Azure AD as the Identity Provider. When a user clicks Login with SSO, our React app redirects them to the Azure AD login page. The user authenticates with their company credentials there — our app never sees the password. Azure AD redirects back to our Express backend with an authorization code. Our backend exchanges that code for an ID token, verifies it, and issues our own JWT for the session. For customers not yet on SSO, we still use our existing JWT username/password flow. SSO is just a different authentication entry point that feeds into the same JWT session system.

---

## Key Things to Remember

**Common Identity Providers:**
- Microsoft Azure Active Directory (Azure AD) — most common in enterprises
- Okta
- Google Workspace
- ADFS — Active Directory Federation Services

**Two SSO protocols:**
- SAML 2.0 — older, enterprise, XML based
- OAuth 2.0 + OIDC (OpenID Connect) — modern, used by Azure AD and Google

**Complete SSO flow:**
1. User clicks Login with SSO on React app
2. Frontend redirects to Identity Provider (Azure AD) login page
3. User enters company credentials on Azure AD — your app never sees the password
4. Azure AD verifies credentials against company directory
5. Azure AD redirects back to your app with an authorization code
6. Express backend exchanges that code for an ID token and access token
7. Backend verifies token, finds or creates user in your DB
8. Backend issues your own JWT for the session
9. From this point your existing JWT flow takes over

**Why SSO is more secure:**
- Your app never handles or stores the user's company password
- If an employee leaves the company, IT disables their Azure AD account and they instantly lose access — no manual deletion needed
- MFA is handled by the IDP automatically — your app gets MFA for free

**SSO vs JWT relationship:**
- SSO = the front door (handles authentication — proving who you are)
- JWT = the room key (handles session — keeping you logged in)
- They do not compete, they work together

---

## Things to Improve
- Confirm with team: which IDP are customers using — Azure AD, Okta, or ADFS?
- Confirm which protocol — SAML or OIDC?
- Read more about OIDC authorization code flow
