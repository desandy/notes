# SameSite Cookies and CSRF

## 1. Quick CSRF Refresher
**Cross-Site Request Forgery (CSRF)** happens when:  
1. You’re logged into a target site (`bank.com`) and have valid session cookies.  
2. You visit an attacker-controlled site (`evil.com`).  
3. That site causes your browser to make a **state-changing request** to `bank.com` (e.g., transferring money).  
4. Because your browser automatically attaches cookies for `bank.com`, the request is **authenticated** — even though you never intended it.

---

## 2. Where SameSite Fits
**SameSite cookies** change whether those cookies get attached in step 3.

### Strict
- Cookies sent **only** for same-site requests.
- CSRF from another site is blocked because cookies aren’t sent at all.

### Lax *(modern default)*
- Cookies **are not** sent for most cross-site requests **unless**:
  - The navigation is *top-level* (clicking a link or submitting a form), **and**
  - The method is **safe** (GET).  
- This blocks most CSRF, because destructive actions usually require POST, PUT, DELETE, etc.
- **Warning:** If your app uses GET for sensitive actions (bad practice), CSRF is still possible.

### None
- Cookies sent with **all** cross-site requests — behaves like pre-SameSite days.
- Must also have `Secure` flag set.
- Vulnerable to CSRF unless other protections (like anti-CSRF tokens) are in place.

---

## 3. How It Blocks CSRF
If SameSite is `Strict` or `Lax` (with POST requests), the browser won’t attach the session cookie when the request originates from a different site.  

Without the session cookie:
- The server sees the request as **unauthenticated**.
- No malicious action can be completed unless the attacker can guess or steal other authentication data.
