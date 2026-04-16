# curl for Website Screening and Technical Validation

A focused, practical guide for checking how a website behaves at the HTTP level.

This is aimed at web reviews, crawler parity tests, redirect validation, bot filtering checks, header inspection, and quick debugging.

## Why curl is useful

`curl` lets you send HTTP requests from the command line and inspect what a server returns without relying on a browser UI. It is especially useful for:

- validating status codes
- checking redirects
- comparing behavior by User-Agent
- confirming whether a bot is blocked
- inspecting headers like `Location`, `Server`, `Set-Cookie`, `Cache-Control`, `Content-Type`
- testing POST requests, forms, APIs, and cookies
- reproducing browser-like requests in a controlled way

## Core concepts

### `-I`
Send a `HEAD` request, which returns headers only.

### `-L`
Follow redirects.

### `-A`
Set the `User-Agent` header.

### `-H`
Send a custom request header.

### `-X`
Set the HTTP method explicitly, like `GET`, `POST`, or `OPTIONS`.

### `-i`
Include response headers with the body.

### `-v`
Verbose mode. Shows the request and connection details.

### `-s`
Silent mode. Useful in scripts.

### `-o`
Write output to a file.

### `-c` and `-b`
Save cookies to a file and send cookies from a file.

### `--max-redirs N`
Limit the number of redirects curl will follow. If the limit is reached, curl exits with an error instead of continuing. Useful for detecting redirect loops.

---

# 1) Basic website screening

## 1. Check the response headers of a page
```bash
curl -I https://example.com/
```
What it does:
- sends a HEAD request
- shows the HTTP status code and response headers
- useful to quickly confirm if the page returns `200`, `301`, `302`, `403`, `404`, `500`, etc.

Use it for:
- validating whether a page is reachable
- checking if a homepage is blocked
- confirming whether a URL is redirecting

## 2. Check a page and follow redirects
```bash
curl -I -L https://example.com/
```
What it does:
- follows the full redirect chain
- shows the final response after redirects

Use it for:
- verifying whether `http` redirects to `https`
- validating `www` to non-`www` or non-`www` to `www`
- checking if a language redirect lands in `/en` or `/fr`

## 3. Show both headers and body
```bash
curl -i https://example.com/
```
What it does:
- returns headers and the response body together

Use it for:
- checking whether a page contains a challenge page, a WAF block page, or an application error

---

# 2) User-Agent and crawler parity tests

## 4. Default request
```bash
curl -I https://example.com/
```
What it does:
- sends curl's default User-Agent

Use it for:
- testing whether generic clients are blocked

## 5. Simulate Googlebot
```bash
curl -I -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://example.com/
```
What it does:
- sends a Googlebot-like `User-Agent`

Use it for:
- checking if the site behaves differently for Googlebot
- detecting user-agent-based filtering

## 6. Simulate Chrome on Windows
```bash
curl -I -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36" https://example.com/
```
What it does:
- mimics a normal desktop browser

Use it for:
- comparing browser-like behavior against generic bot behavior

## 7. Compare full redirect behavior for Googlebot
```bash
curl -I -L -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://example.com/
```
What it does:
- follows all redirects using a Googlebot-like agent

Use it for:
- checking if the redirect chain is stable for a bot

## 8. Compare full redirect behavior for Chrome
```bash
curl -I -L -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36" https://example.com/
```
What it does:
- follows redirects using a browser-like agent

Use it for:
- comparing final destinations between browser and crawler identities

### What to look for in parity testing

If you get different results like these:
- default curl: `403`
- Googlebot: `302`
- Chrome: `200`

then the site is serving different behavior by client identity. That can point to:
- CDN or WAF bot filtering
- edge rules
- load balancer conditions
- origin-side User-Agent logic

---

# 3) Redirect and canonical validation

## 9. Check the `Location` header only
```bash
curl -I https://example.com/
```
What to inspect:
- `Location:`

Use it for:
- confirming where a redirect points
- checking whether `/` goes to `/en`
- checking whether `www` normalizes consistently

## 10. Show every redirect hop in detail
```bash
curl -s -o /dev/null -D - -L https://example.com/
```
What it does:
- suppresses body output
- prints headers for each redirect response

Use it for:
- auditing redirect chains quickly

## 11. Test `www` and non-`www`
```bash
curl -I https://example.com/
curl -I https://www.example.com/
```
What it does:
- compares domain normalization

Use it for:
- checking whether one domain version is canonical
- confirming that both variants do not behave differently

## 12. Test HTTP to HTTPS
```bash
curl -I http://example.com/
curl -I https://example.com/
```
What it does:
- checks transport normalization

Use it for:
- confirming secure redirect behavior

---

# 4) Header inspection

## 13. Inspect server, cache, cookies, and content type
```bash
curl -I https://example.com/
```
What to inspect:
- `server:`
- `cache-control:`
- `content-type:`
- `set-cookie:`
- `x-robots-tag:`
- `location:`

Use it for:
- seeing whether requests are handled by the edge or origin
- checking if cookies are set before redirecting
- confirming if the site marks content noindex via header

## 14. Send a custom header
```bash
curl -I -H "Accept-Language: fr-CA,fr;q=0.9,en;q=0.8" https://example.com/
```
What it does:
- sends a custom `Accept-Language` header

Use it for:
- testing whether the site changes language behavior based on language preference

## 15. Simulate a referrer
```bash
curl -I -e "https://google.com/" https://example.com/
```
What it does:
- sends a `Referer` header

Use it for:
- checking whether the origin treats referral traffic differently

---

# 5) Cookie and session testing

## 16. Save cookies to a file
```bash
curl -c cookies.txt -I https://example.com/
```
What it does:
- stores response cookies in `cookies.txt`

Use it for:
- seeing whether a redirect or challenge depends on cookies

## 17. Reuse cookies in a second request
```bash
curl -b cookies.txt -I https://example.com/
```
What it does:
- sends previously stored cookies

Use it for:
- testing whether access changes after a cookie is set

## 18. Save and reuse cookies in one flow
```bash
curl -c cookies.txt -b cookies.txt -L -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36" https://example.com/
```
What it does:
- behaves more like a simple browser session

Use it for:
- testing language redirects, challenge flows, or session-based redirects

---

# 6) Verbose debugging

## 19. Show connection and TLS details
```bash
curl -v https://example.com/
```
What it does:
- prints connection negotiation details, request headers, response headers, and TLS information

Use it for:
- understanding whether the request reaches the origin
- checking protocol behavior
- inspecting handshake and proxy details

## 20. Save the body to a file while viewing headers
```bash
curl -D headers.txt -o body.html https://example.com/
```
What it does:
- writes headers to `headers.txt`
- writes body to `body.html`

Use it for:
- archiving evidence from a live test
- comparing what different user agents receive

---

# 7) POST, forms, and APIs

## 21. Send a POST request with form data
```bash
curl -X POST -d "email=test@example.com&name=Dario" https://example.com/form-handler
```
What it does:
- submits URL-encoded form data

Use it for:
- testing form endpoints
- reproducing basic form submissions

## 22. Send JSON to an API
```bash
curl -X POST -H "Content-Type: application/json" -d '{"email":"test@example.com","name":"Dario"}' https://api.example.com/endpoint
```
What it does:
- sends a JSON request body

Use it for:
- API testing
- validating JSON request handling

## 23. Include an authorization token
```bash
curl -H "Authorization: Bearer YOUR_TOKEN_HERE" https://api.example.com/endpoint
```
What it does:
- adds a Bearer token

Use it for:
- authenticated API calls

## 24. Send an OPTIONS request for CORS debugging
```bash
curl -i -X OPTIONS \
  -H "Origin: https://example.com" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type, Authorization" \
  https://api.example.com/endpoint
```
What it does:
- simulates a browser preflight request

Use it for:
- testing CORS configuration

---

# 8) Content retrieval and pattern checks

## 25. Download a page to inspect it locally
```bash
curl -L https://example.com/ -o page.html
```
What it does:
- downloads the final response after redirects

Use it for:
- reviewing HTML source
- checking for block pages, challenge pages, or canonical tags

## 26. Search the HTML for canonical tags or strings
```bash
curl -L https://example.com/ | grep -i canonical
```
What it does:
- downloads the page and filters lines containing `canonical`

Use it for:
- quickly checking canonical markup

## 27. Search the HTML for a domain mismatch
```bash
curl -L https://example.com/ | grep -i "www.example.com"
```
What it does:
- scans the HTML for hardcoded domain references

Use it for:
- checking schema or canonical inconsistencies

## 28. Check the HTML lang attribute
```bash
curl -sL -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://example.com/en | grep -i "<html"
```
What it does:
- fetches the page as Googlebot and extracts the opening `<html>` tag
- reveals the declared language of the document

Use it for:
- confirming that `/en` returns `<html lang="en">` and `/fr` returns `<html lang="fr">`
- validating that language handling is reflected in the markup, not just the URL

## 29. Check hreflang tags
```bash
curl -sL -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://example.com/en | grep -i "hreflang"
```
What it does:
- fetches the page and extracts all `hreflang` alternate link tags

Use it for:
- verifying that each language version cross-references the other
- checking whether a self-referencing hreflang and `x-default` are present

## 30. Check robots.txt response headers without following redirects
```bash
curl -sI -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://example.com/robots.txt
```
What it does:
- sends a HEAD request to `/robots.txt` as Googlebot
- does not follow redirects, so you see exactly what that specific path returns

Use it for:
- confirming whether `robots.txt` returns `200` with `content-type: text/plain`
- detecting whether a CDN or edge function intercepts the path and serves something unexpected

## 31. Check redirect loop limit
```bash
curl -sIL --max-redirs 10 -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://example.com/fr
```
What it does:
- follows up to 10 redirects and exits with an error if the limit is reached

Use it for:
- detecting redirect loops between language versions or domain variants
- if curl exits with "too many redirects", a loop exists

## 32. Combine User-Agent and Accept-Language to test language override behavior
```bash
curl -sIL \
  -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  -H "Accept-Language: fr-FR,fr;q=0.9" \
  https://example.com/en
```
What it does:
- sends a Googlebot User-Agent alongside a French language preference header
- tests whether an explicit URL path (`/en`) overrides language detection logic at the CDN or server

Use it for:
- confirming that visiting `/en` keeps the user on English regardless of browser language
- detecting whether Accept-Language-based redirects interfere with explicit language URLs

---

# 9) Timing and availability checks

## 28. Show only the final HTTP code
```bash
curl -s -o /dev/null -w "%{http_code}\n" https://example.com/
```
What it does:
- prints only the response code

Use it for:
- scripts and repeated checks

## 29. Show timing data
```bash
curl -s -o /dev/null -w "code=%{http_code} dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} ttfb=%{time_starttransfer} total=%{time_total}\n" https://example.com/
```
What it does:
- prints key timing values

Use it for:
- spotting slow edge, DNS, TLS, or origin response times

---

# 10) Practical website screening workflows

## Workflow A: Is the homepage blocked?
```bash
curl -I https://example.com/
curl -I -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://example.com/
curl -I -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36" https://example.com/
```
Interpretation:
- if only the generic request is blocked, the site likely filters unknown bots
- if Googlebot gets through but generic bots do not, the site has conditional access

## Workflow B: Is the root redirect deterministic?
```bash
curl -I https://example.com/
curl -I https://www.example.com/
curl -I -L https://example.com/
curl -I -L https://www.example.com/
```
Interpretation:
- check whether all versions converge to one domain and one path

## Workflow C: Does language handling depend on headers or identity?
```bash
curl -I -H "Accept-Language: fr-CA,fr;q=0.9" https://example.com/
curl -I -H "Accept-Language: en-CA,en;q=0.9" https://example.com/
curl -I -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://example.com/
```
Interpretation:
- compare destinations and cookies
- confirm whether behavior changes only by language preference or also by client identity

## Workflow E: Full crawler parity and language stability audit
> Commands used in: **The Canadian Encyclopedia — full test analysis after implementation**
> Client site had been suspended by Google. This workflow was used to verify all fixes were correctly applied.

```bash
# 1. Fetch robots.txt as anonymous client — confirm it is accessible and returns correct content
curl -s https://example.com/robots.txt

# 2. Follow full redirect chain with no User-Agent — baseline behavior for unidentified clients
curl -sIL https://example.com/

# 3. Follow full redirect chain as Googlebot — compare against baseline
curl -sIL -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  https://example.com/

# 4. Follow full redirect chain as Chrome — confirm browsers and Googlebot behave identically
curl -sIL -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36" \
  https://example.com/

# 5. Check robots.txt as Googlebot — headers only, no redirect follow
curl -sI -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  https://example.com/robots.txt

# 6. Check canonical tag as Googlebot — verify canonical is present and points to the correct URL
curl -sL -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  https://example.com/ | grep -i "canonical"

# 7. Verify www redirects to non-www as Googlebot
curl -sIL -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  https://www.example.com/en

# 8. Confirm HTML lang attribute per language version
curl -sL -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  https://example.com/en | grep -i "<html"
curl -sL -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  https://example.com/fr | grep -i "<html"

# 9. Confirm canonical per language version — each should point to itself
curl -sL -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  https://example.com/en | grep -i "canonical"
curl -sL -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  https://example.com/fr | grep -i "canonical"

# 10. Check hreflang tags on each language version
curl -sL -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  https://example.com/en | grep -i "hreflang"
curl -sL -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  https://example.com/fr | grep -i "hreflang"

# 11. Check for redirect loops — if max-redirs is reached, a loop exists
curl -sIL --max-redirs 10 -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  https://example.com/fr

# 12. Confirm explicit language URL ignores Accept-Language header
curl -sIL \
  -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" \
  -H "Accept-Language: fr-FR,fr;q=0.9" \
  https://example.com/en
```

Interpretation:
- Steps 2, 3, 4 compared together detect cloaking — if Googlebot and Chrome get different status codes or redirect destinations than anonymous clients, behavior is conditional
- Step 5 confirms whether a CDN function is intercepting `robots.txt` and serving HTML instead of plain text
- Steps 8–10 validate that language signals in the HTML are consistent with the URL structure
- Step 11 is the loop detection safety net — any result other than a direct `200` with no intermediate redirects warrants investigation
- Step 12 confirms that explicit paths take priority over browser language preference

## Workflow D: Does the final HTML contain canonical mismatches?
```bash
curl -L https://example.com/ | grep -i canonical
curl -L https://example.com/ | grep -i "www.example.com"
curl -L https://example.com/ | grep -i 'application/ld+json'
```
Interpretation:
- check canonical tags and structured-data domain consistency

---

# 11) Useful notes for Windows users

In Git Bash, the examples above usually work as written.

In PowerShell, quoting can differ. For JSON payloads, you may need to escape quotes differently.

Example in PowerShell:
```powershell
curl.exe -I https://example.com/
```

Using `curl.exe` avoids confusion with PowerShell aliases.

---

# 12) Focused sources worth reading

These are the least noisy, most relevant sources for going deeper.

## Official curl sources

### 1. Everything curl
Best deep resource for how curl works and why.
- https://everything.curl.dev/

### 2. curl man page
Best full reference for flags and behaviors.
- https://curl.se/docs/manpage.html

### 3. HTTP scripting with curl
Focused on practical HTTP request work.
- https://curl.se/docs/httpscripting.html

### 4. curl tutorial
Good focused examples for common command usage.
- https://curl.se/docs/tutorial.html

## HTTP behavior references

### 5. MDN: HEAD method
Useful for understanding what `curl -I` is actually doing.
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/HEAD

### 6. MDN: User-Agent header
Useful for understanding why changing `-A` can change server behavior.
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/User-Agent

### 7. MDN: HTTP redirections
Useful for interpreting `301`, `302`, and redirect chains.
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Redirections

### 8. MDN: HTTP headers overview
Useful for quickly understanding what returned headers mean.
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers

---

# 13) A compact starter pack for your own work

These are the 10 commands I would keep closest for website technical screening.

```bash
curl -I https://example.com/
curl -I -L https://example.com/
curl -I -A "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" https://example.com/
curl -I -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36" https://example.com/
curl -I -H "Accept-Language: fr-CA,fr;q=0.9,en;q=0.8" https://example.com/
curl -v https://example.com/
curl -D headers.txt -o body.html https://example.com/
curl -L https://example.com/ | grep -i canonical
curl -s -o /dev/null -w "%{http_code}\n" https://example.com/
curl -s -o /dev/null -w "code=%{http_code} dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} ttfb=%{time_starttransfer} total=%{time_total}\n" https://example.com/
```

---

# 14) Final advice

For website review work, do not rely on a single request.

A good screening habit is:
- test the root URL
- test `www` and non-`www`
- test with default, browser-like, and Googlebot-like agents
- test with and without redirect following
- inspect headers and cookies
- save evidence when behavior changes

That gives you a cleaner picture of whether a site is stable, deterministic, and crawler-safe.
