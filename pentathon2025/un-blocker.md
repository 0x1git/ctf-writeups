
---
## Pentathon 2025 CTF – Un-Blocker \[Easy] \[Web] Write-Up

**Challenge Name:** Un-Blocker
**Category:** Web
**Difficulty:** Easy
**Tags:** SSRF, IP Bypass, URL Filtering, Client-Side Controls

---

### Challenge Description

You're given a minimal website that claims to "Unblock Websites" by acting as a proxy. It takes a URL input, presumably fetches the content server-side, and then renders the response inside an iframe.

The frontend HTML reveals the following details:

* A form with an input for a URL.
* A JavaScript handler that intercepts the form submission and appends the user-supplied URL to `/proxy`.
* The request is made dynamically, and the result is shown in an iframe:

```javascript
const url = new URL("/proxy", window.location.origin);
url.searchParams.append("url", form.url.value);
iframe.src = url;
```

### Analysis

From this behavior, it is clear that the application sends a GET request to `/proxy?url=<user_input>`, which forwards the request server-side and returns the response. This hints at a **Server-Side Request Forgery (SSRF)** vulnerability.

Additionally, there’s a suspicious link in the HTML:

```html
<a href="/flag">Access Restricted Content</a>
```

Accessing `/flag` directly results in a `403 Forbidden` which strongly suggests the flag is meant to be accessed only from the backend.

---

### 🛠️ SSRF Exploitation

When trying `http://127.0.0.1/flag` in the URL field, the backend blocks the request — likely due to a naive blacklist for localhost IPs or domains like `localhost`, `127.0.0.1`, or `::1`.

However, a lesser-known but valid shorthand for localhost in IPv4 is:

```
http://127.1/flag
```
This works because 127.1 is shorthand for 127.0.0.1, as per IPv4 addressing rules. While the backend likely blocked explicit mentions of 127.0.0.1 or localhost, it failed to account for this abbreviated format.
By submitting:

```
http://127.1/flag

```

When submitted via the form, the proxy server fetched /flag from its own loopback interface — and since the request came from the server itself, the flag was revealed.


---

### Final Result

This crafted URL successfully fetched the flag from the internal path and displayed it in the iframe.

> flag{cd3d0356d34630b7b97f972b23a5ee38}

---

### Lessons Learned

* **SSRF Attack Surface:** Any functionality that fetches user-supplied URLs is a potential SSRF vector. Developers must validate outgoing requests carefully.

* **Input Validation Weaknesses:** Blocking certain strings (`127.0.0.1`, `localhost`) isn't sufficient. Attackers can encode IPs in decimal, hexadecimal, octal, or even use DNS rebinding techniques.



