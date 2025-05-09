

## Pentathon 2025 CTF – UnBlocker WriteUp

- **Challenge Name:** Un-Blocker

- **Category:** Web

- **Difficulty:** Easy

- **Tags:** SSRF



### Challenge Description

We are given a minimal website that claims to "Unblock Websites" by acting as a proxy. It takes a URL input, fetches the content, and renders the response inside an iframe.

The frontend HTML reveals the following details:

* A form with an input for a URL.
* A JavaScript handler that intercepts the form submission and appends the user-supplied URL to `/proxy`.
* The request is made dynamically, and the result is shown in an iframe:

```javascript
const url = new URL("/proxy", window.location.origin);
url.searchParams.append("url", form.url.value);
iframe.src = url;
```


From this behavior, it is clear that the application sends a GET request to `/proxy?url=<user_input>`, which forwards the request server-side and returns the response. This hints at a **Server-Side Request Forgery (SSRF)** vulnerability.

Additionally, there’s a suspicious link in the HTML:

```html
<a href="/flag">Access Restricted Content</a>
```

Accessing `/flag` directly results in a `403 Forbidden`, which strongly suggests the flag is meant to be accessed only from the backend.


### SSRF Exploitation

When trying `http://127.0.0.1/flag` in the URL field, the backend blocks the request — likely due to a naive blacklist for localhost IPs or domains like `localhost`, `127.0.0.1`, or `::1`.

However, a lesser-known but valid shorthand for localhost in IPv4 is:

```
http://127.1/flag
```
This works because 127.1 is shorthand for 127.0.0.1, as per IPv4 addressing rules. While the backend blocked 127.0.0.1 and localhost.
However, by submitting:

```
http://127.1/flag
```

The form, the proxy server fetched /flag from its loopback interface, and since the request came from the server itself, the flag was revealed.
> flag{cd3d0356d34630b7b97f972b23a5ee38}


### Lessons Learned

* **SSRF Attack Surface:** Any functionality that fetches user-supplied URLs is a potential SSRF vector. Developers must validate outgoing requests carefully.

* **Input Validation Weaknesses:** Blocking certain strings (`127.0.0.1`, `localhost`) isn't sufficient. Attackers can encode IPs in decimal, hexadecimal, octal, or even use DNS rebinding techniques.



