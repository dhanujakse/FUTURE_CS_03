# FUTURE_CS_03
API Security Risk Analysis Report
Target: JSONPlaceholder REST API · Tool: Postman · Rating: 🔴 CRITICAL (18/100)

Overview
This project is a hands-on API security assessment conducted against the JSONPlaceholder public REST API. The goal was to identify real-world API security vulnerabilities using industry-standard tools and document findings in a professional format aligned with the OWASP API Security Top 10 (2023).
All tests were performed using Postman and Browser DevTools. No systems were harmed — JSONPlaceholder is an intentionally open public demo API used specifically for testing and learning purposes.

What Was Tested
The assessment covered 11 endpoints across the API:
MethodEndpointFindingGET/usersFull PII returned for all users — no auth requiredGET/users/{id}IDOR — any user's profile accessible by IDGET/posts/{id}/commentsEmail addresses exposed in comment objectsPOST/postsXSS and SQL injection payloads accepted without validationPUT/posts/{id}Another user's resource updated — no ownership checkDELETE/posts/{id}Destructive delete accepted — zero authenticationGET/users (×200 rapid)No rate limiting — 200 requests in <5s all returned 200 OKHeaders inspectionAllWildcard CORS, HSTS missing, X-Content-Type-Options missing

Findings Summary
IDTitleSeverityCVSSOWASPAPI-001No Authentication on Any Endpoint🔴 Critical9.1API2:2023API-002Broken Object Level Authorization (IDOR)🔴 Critical8.8API1:2023API-003Excessive Data Exposure — Full PII Returned🔴 Critical7.5API3:2023API-004No Rate Limiting — Brute Force & DDoS Risk🟠 High7.5API4:2023API-005Missing Security Headers (Wildcard CORS, no HSTS)🟠 High6.1API7:2023API-006Unvalidated Input — XSS & SQLi Accepted🟣 Medium6.5API8:2023API-007Verbose Error Messages — Stack Trace Disclosure🟢 Low3.7API7:2023

Key Findings at a Glance
API-001 — No Authentication
Every endpoint is publicly accessible. A DELETE /posts/1 request with no Authorization header returned 200 OK. In a production system, this means any person on the internet can destroy data.
API-002 — IDOR (Broken Object Level Authorization)
Changing the numeric ID in the URL (/users/1, /users/2, /users/5) returns any user's full profile with no ownership verification. This is the #1 most common and most exploited API vulnerability.
API-003 — Excessive PII Disclosure
A single GET /users/1 returns name, username, email, phone number, full physical address, GPS coordinates, and company details — far more than any client needs. This directly violates GDPR's data minimisation principle.
API-004 — No Rate Limiting
Running 200 rapid requests via Postman's Collection Runner returned 200 OK every time. No 429 Too Many Requests, no X-RateLimit-* headers. The entire user database can be scraped in seconds.
API-005 — Missing Security Headers
Access-Control-Allow-Origin: * allows any website to make cross-origin requests on behalf of users. Strict-Transport-Security is absent, enabling HTTP downgrade attacks.
API-006 — No Input Validation
POST /posts with body {"title": "<script>alert(1)</script>", "body": "' OR 1=1 --"} returned 201 Created with the payload echoed back verbatim. No sanitisation, no rejection.

Tools Used

Postman v10 — sending requests, collection runner, header inspection
Browser DevTools — network tab, response analysis
OWASP API Security Top 10 2023 — vulnerability classification framework
CVSS v3.1 — severity scoring


Remediation Priorities

Immediate — Implement JWT / OAuth 2.0 authentication on all endpoints
Immediate — Add per-resource ownership checks to prevent IDOR
Immediate — Filter API responses — return only fields the client actually needs
This Sprint — Add rate limiting at the API gateway; return HTTP 429 with Retry-After
This Sprint — Replace wildcard CORS with explicit origins; add HSTS and security headers
Next Sprint — Add schema validation (Joi / Zod / Pydantic) for all POST and PUT bodies
Next Sprint — Implement a global error handler to suppress stack traces in responses


Skills Demonstrated

API security analysis and manual penetration testing
Authentication and authorization assessment (broken auth, IDOR, access control)
Data exposure risk identification (PII, excessive fields, GDPR relevance)
Security header analysis (CORS, HSTS, CSP, X-Content-Type-Options)
Input validation testing (XSS vectors, SQL injection probing)
Rate limiting and resource consumption testing
Professional security documentation and risk reporting
OWASP API Security Top 10 application
CVSS vulnerability scoring


References

OWASP API Security Top 10 2023
JSONPlaceholder
CVSS v3.1 Calculator
RFC 6749 — OAuth 2.0
GDPR Article 5 — Data Minimisation



Disclaimer: All testing was conducted against JSONPlaceholder, an intentionally open public demo API. No real user data was accessed, no systems were harmed, and all activity was within the intended use of the service. Never perform security testing against systems you do not own or have explicit written permission to test.
