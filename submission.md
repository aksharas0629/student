 Security Report: Reflected XSS Vulnerability in /vulnerable_echo
Author: mataiodoxion
 Vulnerability Summary
The /vulnerable_echo endpoint reflected user input directly into HTML without escaping, allowing a malicious actor to execute arbitrary JavaScript via the browser. This is a classic reflected cross-site scripting (XSS) vulnerability.
Payload Submitted
The following payload was submitted to the /vulnerable_echo endpoint via form input or query string:
<script>alert('xss')</script>
 What Happened
Before the fix:
The page reflected the raw <script> tag directly into the HTML body.
The browser interpreted and executed the script, resulting in an alert popup.
This demonstrated that untrusted input was being interpreted as active code rather than safe text.
� Root Cause
The server embedded the name parameter directly into the HTML response without escaping.
Any <script> tags or other HTML markup from the user were treated as executable code.
 Mitigation Applied
1. Escaped HTML output
User input is now sanitized using markupsafe.escape(name).
Special characters like <, >, and " are rendered as text, preventing execution of injected scripts.
2. Content Security Policy (CSP)
Added a CSP header to disallow external or inline scripts:
Content-Security-Policy: default-src 'self'; script-src 'self'
Provides defense-in-depth even if escaping is bypassed or missed.
Both fixes were implemented in app/server.py. Automated tests confirm that the payload is no longer executed or reflected unsafely.
Test Results (via Pytest)
$ pytest -v --tb=no
================================================================= test session starts =================================================================
platform linux -- Python 3.13.7, pytest-7.4.2, pluggy-1.6.0
rootdir: /home/mataiodoxion/School/CSP/procedures-lab
configfile: pytest.ini
collected 8 items

tests/test_endpoints.py::test_add_endpoint PASSED [ 12%]
tests/test_endpoints.py::test_fib_endpoint PASSED [ 25%]
tests/test_endpoints.py::test_item_crud PASSED [ 37%]
tests/test_endpoints.py::test_vulnerable_echo_reflection SKIPPED (Reflection test ignored because fixed test passes)
tests/test_endpoints.py::test_vulnerable_echo_fixed PASSED [ 62%]
tests/test_utils.py::test_add_simple PASSED [ 75%]
tests/test_utils.py::test_fib_basic PASSED [ 87%]
tests/test_utils.py::test_db_crud PASSED [100%]

============================================================ 7 passed, 1 skipped in 0.02s =============================================================
