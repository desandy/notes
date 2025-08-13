# SameSite Cookie Behavior Cheat Sheet

| Request Type                          | SameSite=Strict           | SameSite=Lax                              | SameSite=None (Secure) |
|---------------------------------------|---------------------------|-------------------------------------------|------------------------|
| **Top-level navigation – GET**        | ❌ Cookies **not sent** if cross-site | ✅ Cookies sent, even if from another site | ✅ Cookies sent         |
| **Top-level navigation – POST**       | ❌ Cookies **not sent**    | ❌ Cookies **not sent**                    | ✅ Cookies sent         |
| **Iframe load – any method**           | ❌ Cookies **not sent**    | ❌ Cookies **not sent**                    | ✅ Cookies sent         |
| **AJAX / fetch – any method**          | ❌ Cookies **not sent**    | ❌ Cookies **not sent**                    | ✅ Cookies sent         |
| **Image / script / CSS request**       | ❌ Cookies **not sent**    | ❌ Cookies **not sent**                    | ✅ Cookies sent         |
| **Form submission (POST) from another site** | ❌ Cookies **not sent**    | ❌ Cookies **not sent**                    | ✅ Cookies sent         |
| **Same-site navigation (any method)**  | ✅ Cookies sent            | ✅ Cookies sent                            | ✅ Cookies sent         |

---

## Legend:
- ✅ Cookies sent
- ❌ Cookies not sent

---

## Notes:
1. **Top-level navigation** = The main browser URL changes (clicking a link, submitting a form to `_top`, or setting `window.location`).
2. **Safe method** = HTTP method considered safe by HTTP spec (GET, HEAD, OPTIONS, TRACE). For SameSite purposes, only GET is relevant.
3. **SameSite=None** requires the `Secure` flag and HTTPS.
4. SameSite is enforced **per cookie** — you can set different rules for different cookies.

---

## CSRF Implications:
- **Strict**: Strongest protection, but may break legitimate cross-site flows (e.g., login redirects).
- **Lax**: Blocks most CSRF, except unsafe GET actions triggered by top-level navigation.
- **None**: Vulnerable to CSRF unless other defenses (like CSRF tokens) are implemented.

# SameSite Cookies vs. CSRF Tokens

## 1. What SameSite Cookies Do
- **Browser-side control**: The browser decides whether to send cookies in a cross-site request based on the cookie’s `SameSite` attribute.
- **Strengths**:
  - No server code needed (just set the attribute).
  - Blocks many CSRF attempts automatically.
- **Weaknesses**:
  - Doesn’t protect if `SameSite=None` or misconfigured.
  - Allows certain cross-site requests in `Lax` mode (top-level GET).
  - No effect if site uses non-cookie credentials (e.g., `Authorization` header).

---

## 2. What CSRF Tokens Do
- **Server-side validation**: The server embeds a unique, unpredictable token in every state-changing form/page, and verifies it on submission.
- **Strengths**:
  - Works regardless of cookie settings.
  - Protects even if cookies are always sent (`SameSite=None`).
  - Flexible — can defend against cross-site requests with any HTTP method.
- **Weaknesses**:
  - Requires server-side code and token management.
  - Needs correct implementation (predictable/static tokens can be bypassed).
  - Vulnerable if attacker can read the page (via XSS).

---

## 3. How They Compare

| Feature                  | SameSite Cookies                | CSRF Tokens                     |
|--------------------------|----------------------------------|-----------------------------------|
| **Where enforced**       | Browser                         | Server                           |
| **Setup complexity**     | Very low                        | Higher                           |
| **Blocks all CSRF?**     | No — `Lax` allows some GETs      | Yes (if implemented correctly)   |
| **Dependent on cookies?**| Yes                              | No (works with any auth method)  |
| **XSS impact**           | Safe from XSS? ✅ (mostly)       | ❌ Token can be stolen via XSS    |

---

## 4. Best Practice
- **Use both**:
  1. Set cookies with `SameSite=Lax` or `Strict`.
  2. Use anti-CSRF tokens for all state-changing requests.
- **Reason**:
  - SameSite provides a passive, browser-enforced baseline defense.
  - CSRF tokens provide active, server-enforced verification.

---

## ✅ Mini-check
If a site uses `SameSite=Strict` session cookies but **no CSRF token**, can a top-level GET request from another site perform a money transfer? Why or why not?

