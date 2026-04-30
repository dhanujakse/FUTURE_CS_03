# API Security Risk Analysis Report
### Target: JSONPlaceholder REST API · Tool: Postman · Rating: 🔴 CRITICAL (18/100)

---

## Overview

This project is a hands-on API security assessment conducted against the [JSONPlaceholder](https://jsonplaceholder.typicode.com) public REST API. The goal was to identify real-world API security vulnerabilities using industry-standard tools and document findings in a professional format aligned with the OWASP API Security Top 10 (2023).

All tests were performed using **Postman** and **Browser DevTools**. No systems were harmed — JSONPlaceholder is an intentionally open public demo API used specifically for testing and learning purposes.

---

## What Was Tested

The assessment covered 11 endpoints across the API:

| Method | Endpoint | Finding |
|--------|----------|---------|
| GET | `/users` | Full PII returned for all users — no auth required |
| GET | `/users/{id}` | IDOR — any user's profile accessible by ID |
| GET | `/posts/{id}/comments` | Email addresses exposed in comment objects |
| POST | `/posts` | XSS and SQL injection payloads accepted without validation |
| PUT | `/posts/{id}` | Another user's resource updated — no ownership check |
| DELETE | `/posts/{id}` | Destructive delete accepted — zero authentication |
| GET | `/users` (×200 rapid) | No rate limiting — 200 requests in <5s all returned 200 OK |
| Headers inspection | All | Wildcard CORS, HSTS missing, X-Content-Type-Options missing |

---

## Findings Summary

| ID | Title | Severity | CVSS | OWASP |
|----|-------|----------|------|-------|
| API-001 | No Authentication on Any Endpoint | 🔴 Critical | 9.1 | API2:2023 |
| API-002 | Broken Object Level Authorization (IDOR) | 🔴 Critical | 8.8 | API1:2023 |
| API-003 | Excessive Data Exposure — Full PII Returned | 🔴 Critical | 7.5 | API3:2023 |
| API-004 | No Rate Limiting — Brute Force & DDoS Risk | 🟠 High | 7.5 | API4:2023 |
| API-005 | Missing Security Headers (Wildcard CORS, no HSTS) | 🟠 High | 6.1 | API7:2023 |
| API-006 | Unvalidated Input — XSS & SQLi Accepted | 🟣 Medium | 6.5 | API8:2023 |
| API-007 | Verbose Error Messages — Stack Trace Disclosure | 🟢 Low | 3.7 | API7:2023 |

---

## Key Findings at a Glance

**API-001 — No Authentication**
Every endpoint is publicly accessible. A `DELETE /posts/1` request with no `Authorization` header returned `200 OK`. In a production system, this means any person on the internet can destroy data.

**API-002 — IDOR (Broken Object Level Authorization)**
Changing the numeric ID in the URL (`/users/1`, `/users/2`, `/users/5`) returns any user's full profile with no ownership verification. This is the #1 most common and most exploited API vulnerability.

**API-003 — Excessive PII Disclosure**
A single `GET /users/1` returns name, username, email, phone number, full physical address, GPS coordinates, and company details — far more than any client needs. This directly violates GDPR's data minimisation principle.

**API-004 — No Rate Limiting**
Running 200 rapid requests via Postman's Collection Runner returned `200 OK` every time. No `429 Too Many Requests`, no `X-RateLimit-*` headers. The entire user database can be scraped in seconds.

**API-005 — Missing Security Headers**
`Access-Control-Allow-Origin: *` allows any website to make cross-origin requests on behalf of users. `Strict-Transport-Security` is absent, enabling HTTP downgrade attacks.

**API-006 — No Input Validation**
`POST /posts` with body `{"title": "<script>alert(1)</script>", "body": "' OR 1=1 --"}` returned `201 Created` with the payload echoed back verbatim. No sanitisation, no rejection.

---

## Tools Used

- **Postman v10** — sending requests, collection runner, header inspection
- **Browser DevTools** — network tab, response analysis
- **OWASP API Security Top 10 2023** — vulnerability classification framework
- **CVSS v3.1** — severity scoring

---

## Remediation Priorities

1. **Immediate** — Implement JWT / OAuth 2.0 authentication on all endpoints
2. **Immediate** — Add per-resource ownership checks to prevent IDOR
3. **Immediate** — Filter API responses — return only fields the client actually needs
4. **This Sprint** — Add rate limiting at the API gateway; return `HTTP 429` with `Retry-After`
5. **This Sprint** — Replace wildcard CORS with explicit origins; add HSTS and security headers
6. **Next Sprint** — Add schema validation (Joi / Zod / Pydantic) for all POST and PUT bodies
7. **Next Sprint** — Implement a global error handler to suppress stack traces in responses

---

## Skills Demonstrated

- API security analysis and manual penetration testing
- Authentication and authorization assessment (broken auth, IDOR, access control)
- Data exposure risk identification (PII, excessive fields, GDPR relevance)
- Security header analysis (CORS, HSTS, CSP, X-Content-Type-Options)
- Input validation testing (XSS vectors, SQL injection probing)
- Rate limiting and resource consumption testing
- Professional security documentation and risk reporting
- OWASP API Security Top 10 application
- CVSS vulnerability scoring

---

## References

- [OWASP API Security Top 10 2023](https://owasp.org/www-project-api-security/)
- [JSONPlaceholder](https://jsonplaceholder.typicode.com)
- [CVSS v3.1 Calculator](https://nvd.nist.gov/vuln-metrics/cvss)
- [RFC 6749 — OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749)
- [GDPR Article 5 — Data Minimisation](https://gdpr-info.eu/art-5-gdpr/)

---

> **Disclaimer:** All testing was conducted against JSONPlaceholder, an intentionally open public demo API. No real user data was accessed, no systems were harmed, and all activity was within the intended use of the service. Never perform security testing against systems you do not own or have explicit written permission to test.
