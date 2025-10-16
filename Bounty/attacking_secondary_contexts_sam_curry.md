# Attacking Secondary Contexts in Web Applications

## Overview

- Focuses on how web requests pass through secondary contexts such as proxies, CDNs, and internal APIs, and how attackers can exploit those layers.
- Highlights discovery techniques, example attack chains, and a real-world bypass of Authy two-factor authentication.
- Serves as a reference checklist for identifying routing quirks, validation gaps, and traversal opportunities.

## Presenter

- Sam Curry (@samwcyo)
- Full-time bug bounty hunter (three years on and off)
- Application security researcher and blogger at [samcurry.net](https://samcurry.net)

## Glossary

- **Secondary context:** Any backend service, proxy, CDN, or API that processes a user's request after the initial web server.
- **Directory traversal:** Manipulating URL paths (for example, using `../`) to reach unintended directories or services.
- **IDOR (Insecure Direct Object Reference):** Accessing objects (files, records) without proper authorization checks.
- **Request smuggling:** Crafting HTTP requests so that intermediaries interpret message boundaries differently.
- **CRLF injection:** Inserting carriage return and line feed characters to tamper with HTTP headers or responses.
- **Max-Forwards header:** An HTTP header that limits proxy hops, useful for tracing request paths.

## From Simple Servers to Complex Routing

Traditional web servers map routes directly to files on disk. For example, `GET /index.html` resolves to `/var/www/html/index.html`. Modern applications rarely stay that simple, though. Requests travel through middleware, proxies, and load balancers, and responses may come from code-defined routes or entirely different hosts. Understanding those additional layers is the first step toward finding bugs in secondary contexts.

## Why Secondary Contexts Matter

- Backend requests often use different authentication models, hosts, or validation rules.
- User-controlled data can be reused deeper in the stack, enabling request smuggling, CRLF injection, open redirects, SSRF, or XSS.
- Developers seldom expect attackers to control internal paths, so debugging or administrative features (for example, `?debug=1` or `/server-status`) can leak through.

## Investigation Checklist

1. Map routing layers: identify CDNs, reverse proxies, API gateways, and internal hosts.
2. Probe path handling: attempt traversal (`../`), alternate encodings, and double encoding to test canonicalization.
3. Compare responses: note header changes, status codes, and content types across directories and hostnames.
4. Capture errors: look for stack traces, upstream hostnames, or leaked credentials in failure responses.
5. Trace authentication: determine where cookies, tokens, or headers are forwarded and whether secondary contexts enforce access control.
6. Escalate impact: test whether traversal enables IDOR, file disclosure, or SSRF, and chain findings across related endpoints.

## Techniques for Spotting Hidden Routing

- **Directory traversal:** Compare responses for `/api/../` versus `/` to see whether routing changes.
- **Encoding tricks:** Send encoded control characters such as `%23`, `%3F`, `%26`, `%2E`, `%2F`, and `%40`, and try double or triple encoding.
- **Header comparisons:** Watch for shifts in headers, status codes, or content types between endpoints such as `/` and `/images/`.
- **Error leakage:** Collect upstream error messages like `internal.company.com:8080 returned 500 Internal Server Error` to learn about internal hosts.

## Field Notes and Examples

- **Static asset proxy pivot:** A request for `/favicon.ico` might be served by CloudFront. Traversing with `/favicon.ico/..%2f..%2fattackersbucket%2fxss.html` reveals whether the CDN forwards requests to an origin such as an S3 bucket (`https://s3.amazonaws.com/yahoo-bucket/favicon.ico/../../attackersbucket/xss.html`), enabling stored XSS or content injection.
- **Segmented API discovery:** Visiting `/api/v1/` can return different headers and behavior than the web root. Attempting `/api/v1/../../../` shows whether the API runs on a separate backend that can be reached via traversal.
- **Encoding-induced failure:** If `%23` becomes `#` and causes the upstream request to drop parameters, the secondary context likely rebuilds the URL differently, giving attackers leverage over query strings or fragments.
- **Session-bound traversal:** Overwriting API paths allows attackers to pivot from their own session identifier to a victim's and read files with requests like `GET /files/..%2f..%2f<victim-id>%2f<filename>`.
- **Lack of normalization:** Some APIs never canonicalize paths, so traversal continues to work against services assumed to be inaccessible from the internet.
- **Invoice retrieval chain:** `GET /my-services/invoices/..%2finvoices%2fINV08179455/pdf` returns a PDF, while deeper traversal yields 404. The mismatch hints at an upstream service with a unique directory layout worth investigating.
- **Identifier leakage:** An unrelated error message (`Id samwcurry@gmail.com#vj does not have permission...`) exposes the directory naming scheme. Knowing a user's email and invoice number lets an attacker fetch any invoice.
- **Escalating to payment data:** The same traversal techniques applied to `/subscriptions/:id` and `/paymentmethods/:id` reveal payment method data. Brute-forcing a subscription identifier exposes the payment method directory, so possessing only the victim's email address leads to sensitive financial information.

## Broader Exploration

Directory traversal is not the only path to impact. Some internal APIs act like Boolean queries, returning `200 OK` for "record exists" and `404` otherwise, so careful probing can map internal resources without direct traversal. The real skill lies in tracing every hop a request takes and predicting how each service processes user-supplied data.

## Case Study: Authy 2FA Bypass

- **Context:** Authy ships as a client library, creating the chain `User -> Client -> Authy`.
- **Root cause:** The client trusted any response that returned `HTTP 200 OK` alongside `{"success":true}`.
- **Key code path:**

```javascript
this._request("get", "/protected/json/verify/" + token + "/" + id, {}, callback, qs);
```

Requesting `/protected/json` directly returned the same `200` response with `{"success":true}`. That single oversight yielded a universal 2FA bypass for many Authy integrations (credit: Egor Homakov, @homakov).

## Defensive Considerations

- Normalize and validate paths at every proxy hop; reject ambiguous encodings before forwarding requests.
- Enforce authorization checks in each context rather than assuming upstream enforcement.
- Limit internal feature exposure by gating debug endpoints, admin paths, and verbose error messages.
- Instrument logging to record traversal attempts, malformed encodings, and Max-Forwards usage.
- Regularly audit dependency libraries (such as 2FA clients) for implicit trust assumptions about upstream services.

## Key Takeaways

- Secondary contexts expose additional attack surface because internal services interpret user input differently.
- Authorization and validation often diverge across layers, so a `200` in one context can bypass the intended `403` in another.
- Developers face a complex problem: multiple services with distinct behaviors and sanitization requirements spread across two or three proxies. Research such as using the `Max-Forwards` header for tracing requests ([agarri.fr](https://www.agarri.fr/blog/archives/2011/11/12/traceroute-like_http_scanner/index.html)) highlights how hard the problem is to solve.

## References

- Sam Curry, "Attacking Secondary Contexts in Web Applications," Kernelcon talk.
- Egor Homakov (@homakov) disclosure on Authy 2FA bypass.
- [https://www.agarri.fr/blog/archives/2011/11/12/traceroute-like_http_scanner/index.html](https://www.agarri.fr/blog/archives/2011/11/12/traceroute-like_http_scanner/index.html) - Using Max-Forwards for HTTP traceroute.
