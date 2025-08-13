# Same Origin Policy (SOP) and SameSite Cookies

## 1. Same Origin Policy (SOP) Basics
**Definition:**  
The Same Origin Policy is a browser security rule that says:  
> *A web page can only read data from another page if both pages have the same **origin**.*

**Origin** means:  
scheme (protocol) + host (domain) + port


**Examples:**
- ✅ Same origin:  
  `https://example.com:443/page1` → `https://example.com:443/page2`
- ❌ Different origin:  
  - Different domain: `https://example.com` vs `https://other.com`  
  - Different protocol: `http://example.com` vs `https://example.com`  
  - Different port: `https://example.com:443` vs `https://example.com:8443`

**Why it exists:**  
Without SOP, malicious sites could read your private banking data just because you’re logged in somewhere else.

---

## 2. What SOP Restricts
- **JavaScript** access to:
  - DOM of another origin
  - `window` objects of other origins
- **XHR / fetch** calls:
  - You can send requests cross-origin, but you *cannot* read the responses unless the server allows it via CORS.
- SOP does **not** block sending requests — just reading responses.

---

## 3. Cookies and SOP
- SOP is about **reading data** in JavaScript.
- Cookies can still be *sent* cross-origin under certain conditions — even if JS can’t read them.
- Example: If you’re on `evil.com` and your browser requests something from `bank.com`, your browser might attach your `bank.com` cookies automatically (unless prevented).

---

## 4. SameSite Cookies
**Purpose:** Prevent cookies from being sent in *cross-site* requests unless intended.  
- Introduced to reduce CSRF attacks.

**SameSite options:**
1. **Strict**  
   - Cookies sent **only** when navigating from the same site.  
   - Even clicking a link from another site won’t send them.
2. **Lax** *(default in modern browsers)*  
   - Cookies sent for *top-level* navigations (like clicking a link) but not for things like iframes or AJAX requests from other sites.
3. **None**  
   - Cookies sent in all requests — but must have `Secure` flag set (HTTPS only).

---

## 5. How They Fit Together
- **SOP** stops JS from reading sensitive data cross-origin.
- **SameSite** controls whether cookies are even *sent* in cross-site requests.
- They address **different risks**:
  - SOP → Prevents *data leaks* to untrusted origins.
  - SameSite → Prevents *automatic credential use* in cross-site requests.

---


