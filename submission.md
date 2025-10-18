What I implemented
Implemented small utility procedures in app/utils.py:
add(a, b) — returns the sum of two floats.
fib(n) — iterative Fibonacci implementation with fib(0) == 0, fib(1) == 1.
Simple in-memory DB operations using _DB: Dict[str, Dict]:
create_item(key, value) stores a deep copy of value under key.
read_item(key) returns a deep copy of the stored dict or None if absent.
update_item(key, patch) merges patch into the stored dict (existing.update(patch)) and returns True if the key existed.
delete_item(key) deletes the key and returns True if deleted.
clear_db() resets the database for tests.
Made a small server change in app/server.py for security:
Replaced the vulnerable echo with an escaped version using markupsafe.escape(name) so user-supplied input is HTML-escaped before insertion.
Added a basic Content-Security-Policy header (applied to text/html responses) to disallow inline scripts as an additional mitigation.
Discovery (what I sent and what happened)
name=<script>alert(1)</script>
# URL encoded form: ?name=%3Cscript%3Ealert(1)%3C%2Fscript%3E
What happened in the browser (during discovery, before the fix):
The server reflected the payload into the HTML response unescaped, producing a body such as:
<h2>Hello <script>alert(1)</script></h2>
Why the server was vulnerable
The /vulnerable_echo endpoint reflected untrusted user input directly into HTML without escaping.
The browser treats <script>...</script> as executable JavaScript when present in an HTML response, resulting in reflected Cross-Site Scripting (XSS).
Mitigation applied
Escaping
Replaced raw insertion with escape(name) from MarkupSafe.
Converts < and > (and other special characters) into their HTML entity equivalents (e.g., &lt; and &gt;), so the browser renders the tags as text rather than executing them.
Content Security Policy (CSP)
Added a basic header to HTML responses via @app.after_request: Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self';
Prevents inline scripts and loading scripts from remote origins by default. Provides defense-in-depth and helps mitigate XSS even if an escape step is missed or bypassed.
Verification / Tests
Ran the project test suite with pytest -q.
Current result (local): 7 passed, 1 skipped
Specific expectations per the lab:
test_vulnerable_echo_reflection (the test that checks raw reflection) is skipped/ignored because the fixed test passes.
test_vulnerable_echo_fixed (the test that asserts the raw <script> no longer appears) passes.
How to reproduce locally
# Create and activate the virtualenv
python -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run tests
pytest -q

# Run the server and open the forms in a browser
./run.sh
# visit http://127.0.0.1:8000/

# Optional: check the CSP header on HTML responses
curl -I http://127.0.0.1:8000/ | grep -i Content-Security-Policy
Full pytest output (verbose)
================================= test session starts =================================
platform linux -- Python 3.12.3, pytest-7.4.2, pluggy-1.6.0 -- /home/adityasri/nighthawk/Aditya_2026/venv/bin/python
cachedir: .pytest_cache
rootdir: /home/adityasri/nighthawk/procedures-lab
configfile: pytest.ini
plugins: anyio-4.10.0
collected 8 items

tests/test_endpoints.py::test_add_endpoint PASSED                               [ 12%]
tests/test_endpoints.py::test_fib_endpoint PASSED                               [ 25%]
tests/test_endpoints.py::test_item_crud PASSED                                  [ 37%]
tests/test_endpoints.py::test_vulnerable_echo_reflection SKIPPED (Reflectio...) [ 50%]
tests/test_endpoints.py::test_vulnerable_echo_fixed PASSED                      [ 62%]
tests/test_utils.py::test_add_simple PASSED                                     [ 75%]
tests/test_utils.py::test_fib_basic PASSED                                      [ 87%]
tests/test_utils.py::test_db_crud PASSED                                        [100%]

============================ 7 passed, 1 skipped in 0.04s =============================
