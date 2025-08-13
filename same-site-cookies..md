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
4. SameSite i
