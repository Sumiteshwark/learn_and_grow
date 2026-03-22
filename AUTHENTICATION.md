**Last Updated:** March 2026 
**Author:** SUMITESHWAR KUMAR

---

# Authentication & Authorization — Industry-Level Reference

> Complete reference of authentication concepts, flows, and patterns as implemented in industry.
> Based on the BPSC Quiz App implementation and discussions.

---

## Table of Contents

1. [JWT Authentication Flow](#1-jwt-authentication-flow)
2. [Cookie-Based Token Storage](#2-cookie-based-token-storage)
3. [Refresh Token Pattern](#3-refresh-token-pattern)
4. [Where Data Lives](#4-where-data-lives)
5. [Session Expiry & Silent Refresh](#5-session-expiry--silent-refresh)
6. [Rate Limiting](#6-rate-limiting)
7. [Reverse Proxy vs Rate Limiter vs API Gateway](#7-reverse-proxy-vs-rate-limiter-vs-api-gateway)
8. [RBAC — Role-Based Access Control](#8-rbac--role-based-access-control)
9. [Role Assignment on Registration](#9-role-assignment-on-registration)
10. [Cross-Origin Cookie Issues & Proxy Fix](#10-cross-origin-cookie-issues--proxy-fix)

---

## 1. JWT Authentication Flow

### What is JWT?

JSON Web Token — a signed string that proves identity without server-side session storage.

```
Structure:  header.payload.signature

Header:    { "alg": "HS256", "typ": "JWT" }
Payload:   { "userId": "abc123", "role": "student", "iat": 1773986916, "exp": 1773990516 }
Signature: HMACSHA256(header + "." + payload, JWT_SECRET)
```

### The Flow

```
1. LOGIN
   Client sends: { email, password }
   Server: validates → generates JWT → sets cookie
   JWT payload: { userId, role, iat, exp }

2. SUBSEQUENT REQUESTS
   Browser auto-sends cookie with every request
   Server: extracts JWT from cookie → verifies signature → extracts userId
   No database lookup for auth (stateless) — only to get user details

3. TOKEN EXPIRY
   JWT has built-in expiry (exp field)
   After expiry: jwt.verify() throws TokenExpiredError
   Client must refresh or re-login
```

### Why JWT (not session-based)?

```
Session-based:
  Server stores session in memory/DB
  Cookie contains session ID (not data)
  Every request → DB lookup to validate session
  Problem: doesn't scale across multiple servers without shared session store

JWT-based:
  Server stores nothing (stateless)
  Cookie contains the actual user data (signed)
  Every request → verify signature (no DB lookup)
  Scales across any number of servers
```

---

## 2. Cookie-Based Token Storage

### Why Cookies (not localStorage)?

```
localStorage:
  ✅ Easy to use (localStorage.setItem)
  ❌ Accessible by JavaScript → XSS attack can steal token
  ❌ Must manually attach to every request

httpOnly Cookie:
  ✅ NOT accessible by JavaScript → immune to XSS
  ✅ Auto-attached to every request by browser
  ✅ Can set Secure (HTTPS only), SameSite, expiry
  ❌ Slightly more complex to set up
```

### Cookie Attributes

```
res.cookie('access_token', jwt, {
  httpOnly: true,        // JS can't read it (XSS protection)
  secure: true,          // Only sent over HTTPS
  sameSite: 'lax',       // Same-origin: 'lax', cross-origin: 'none'
  maxAge: 3600000,       // 1 hour in milliseconds
  path: '/',             // Sent for all routes
});
```

### SameSite Explained

```
'strict':  Cookie NEVER sent cross-origin. Even clicking a link from email won't send it.
'lax':     Cookie sent on navigation (clicking links) but not on AJAX/fetch cross-origin.
'none':    Cookie always sent cross-origin. REQUIRES secure: true. Used for cross-domain APIs.
```

### Same-Origin vs Cross-Origin

```
Same-origin (cookie works easily):
  Frontend: https://myapp.com
  Backend:  https://myapp.com/api
  → sameSite: 'lax' works fine

Cross-origin (cookie needs special config):
  Frontend: https://app.vercel.app
  Backend:  https://api.render.com
  → sameSite: 'none', secure: true
  → Modern browsers may STILL block (third-party cookie blocking)
  → Solution: proxy through same domain (see section 10)
```

---

## 3. Refresh Token Pattern

### The Problem

```
Single JWT (1 hour expiry):
  User takes a 30-min quiz
  Token expires mid-quiz
  Quiz submission fails → user must re-login → quiz lost
```

### The Solution: Two Tokens

```
Access Token:
  Short-lived: 1 hour
  Stored in: httpOnly cookie
  Used for: every API call
  Stateless: no DB lookup to validate

Refresh Token:
  Long-lived: 7 days
  Stored in: httpOnly cookie + database
  Used for: getting a new access token when it expires
  Stateful: validated against DB (can be revoked)
```

### How It Works

```
LOGIN:
  → Generate access token (1h) → set cookie
  → Generate refresh token (7d) → set cookie + store in DB

NORMAL API CALL:
  → Send access token cookie → verify → respond

ACCESS TOKEN EXPIRES (after 1 hour):
  → API returns 401 { code: 'TOKEN_EXPIRED' }
  → Frontend calls POST /api/auth/refresh
  → Server validates refresh token against DB
  → Deletes old refresh token (rotation)
  → Issues new access token + new refresh token
  → Frontend retries original request → succeeds
  → User never notices

REFRESH TOKEN EXPIRES (after 7 days of inactivity):
  → Refresh attempt fails
  → User must re-login

LOGOUT:
  → Delete refresh token from DB (revoked)
  → Clear both cookies
  → Old access token still valid for up to 1 hour (stateless)
  → But refresh is gone → no renewal possible
```

### Refresh Token Rotation

```
Every time refresh token is used:
  Old refresh token → DELETED from DB
  New refresh token → CREATED in DB + set in cookie

Why: If attacker steals a refresh token and you use it first,
     attacker's stolen token no longer exists → attack blocked.
```

### One Token Per User (vs Multi-Device)

```
Option A — One per user (simpler):
  Login from any device → deletes old refresh token → creates new one
  Previous device's refresh fails → must re-login
  Logout = delete the one token → effectively logs out everywhere

Option B — One per device (multi-device support):
  Each login creates its own refresh token
  Multiple active tokens in DB per user
  Logout deletes only the current device's token
  Need "logout all devices" to clear all
```

---

## 4. Where Data Lives

```
┌─────────────────────────────────────────────────┐
│                  Browser                         │
│                                                  │
│  Cookies (httpOnly, auto-sent to server):         │
│  ├── access_token:  JWT (1h expiry)              │
│  └── refresh_token: random string (7d expiry)    │
│                                                  │
│  localStorage (persisted, JS-accessible):         │
│  └── quiz-storage: { answers, timer, chapter }   │
│                                                  │
│  sessionStorage (dies when tab closes):           │
│  └── redirectAfterLogin: "/chapter/DBMS"         │
│                                                  │
│  Memory (Zustand stores, lost on refresh):        │
│  ├── timerStore: { timeElapsed, isRunning }      │
│  └── API caches: topics, subtopics               │
│                                                  │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│                  Server (MongoDB)                 │
│                                                  │
│  User:          { name, email, password, role }  │
│  RefreshToken:  { userId, token, expiresAt }     │
│  QuizSession:   { userId, startTime, status }    │
│  Attempt:       { answers, results, timeTaken }  │
│                                                  │
└─────────────────────────────────────────────────┘
```

### What Survives What

```
Event              │ Cookies  │ localStorage  │ sessionStorage  │ Memory  │ DB
───────────────────┼──────────┼───────────────┼─────────────────┼─────────┼─────
Page refresh       │ ✅ stays │ ✅ stays      │ ✅ stays        │ ❌ lost │ ✅
Close tab          │ ✅ stays │ ✅ stays      │ ❌ lost         │ ❌ lost │ ✅
Close browser      │ ✅ stays │ ✅ stays      │ ❌ lost         │ ❌ lost │ ✅
Logout             │ ❌ cleared│ ❌ cleared   │ ❌ cleared      │ ❌ lost │ ✅
Cookie expires     │ ❌ gone  │ ✅ stays      │ ✅ stays        │ ✅ stays│ ✅
Clear site data    │ ❌ gone  │ ❌ gone       │ ❌ gone         │ ❌ lost │ ✅
```

---

## 5. Session Expiry & Silent Refresh

### Frontend 401 Handling

```
API call returns 401
  ↓
Check error code:
  ├── TOKEN_EXPIRED → attempt silent refresh → retry request
  ├── TOKEN_INVALID → force logout, redirect to login
  └── NO code       → force logout, redirect to login

Silent refresh:
  POST /api/auth/refresh → success? → retry original request (user never notices)
                         → failed?  → clear state, redirect to login
```

### Race Condition: Multiple 401s

```
Problem:
  3 parallel API calls → all return 401 → all try to redirect to login
  Result: multiple redirects, state corruption

Solution: Lock mechanism
  let isSessionExpired = false;

  On 401: if (!isSessionExpired) {
    isSessionExpired = true;
    // clear state, redirect — only happens once
  }
```

### Periodic Auth Validation

```
Safety net for idle users:
  setInterval(checkAuth, 5 * 60 * 1000);  // every 5 minutes

If user is idle (no API calls) and token expires:
  Periodic check detects it → redirect to login
  Without this: user stays on page, sees broken UI on next action
```

### Post-Login Redirect

```
Session expires on /chapter/DBMS
  → Save path: sessionStorage.setItem('redirectAfterLogin', '/chapter/DBMS')
  → Redirect to /login
  → User logs in
  → Read saved path → navigate to /chapter/DBMS (not /)
  → Remove from sessionStorage
```

---

## 6. Rate Limiting

### Why Rate Limiting?

```
Without it:
  Attacker sends 10,000 login attempts/second → brute force passwords
  Scraper downloads entire question bank in minutes
  Single user overwhelms server → other users get slow responses
```

### Strategies (Ordered by Complexity)

#### Strategy 1: Global Rate Limit

```
Every IP → 100 requests per minute across ALL routes
Safety net. Catches basic abuse.
```

#### Strategy 2: Tiered Route Limits

```
Auth routes (most attacked):    5 req / 15 min per IP
Write routes (expensive):       10 req / 15 min per user
Read routes (cheap, cached):    60 req / 1 min per user
Open routes (/health):          No limit
```

#### Strategy 3: Per-User vs Per-IP

```
Per-IP problem:
  500 students on university Wi-Fi share one IP
  One student hits limit → all 500 blocked

Per-User solution:
  keyGenerator: (req) => req.user?.id || req.ip
  Each student gets their own limit
```

#### Strategy 4: Progressive Delay (instead of lockout)

```
Per IP+email combination:
  Fail 1-3:    Instant response
  Fail 4-5:    2 second delay
  Fail 6-7:    5 second delay
  Fail 8-10:   10 second delay
  Fail 11+:    30 second delay
  Success:     Reset counter

Real user from different IP → instant (not affected by attacker)
Account NEVER locks → real user can always login

Why not hard lockout?
  Attacker knows your email → intentionally fails 10 times → YOUR account locked
  Progressive delay slows attacker without hurting real user
```

#### Strategy 5: Fixed Window vs Sliding Window

```
Fixed Window (express-rate-limit default):
  Window: [00:00 ─── 00:15] [00:15 ─── 00:30]
  10 requests at 00:14, 10 more at 00:16 = 20 in 2 minutes (bypass!)

Sliding Window (Redis-backed):
  At any point, look back exactly 15 minutes
  Always accurate. No boundary exploit.
```

#### Strategy 6: Token Bucket (AWS, Google)

```
Bucket: 10 tokens, refills 1/second
  → Allows bursts (up to 10 at once)
  → But averages out to 1 req/sec over time

Real user:   Opens app → 8 quick calls (burst) → then idle → fine
Attacker:    Constant stream → bucket drains → blocked
```

### Single Server vs Multi-Server

```
Single server (in-memory):
  Counters in Node.js process memory
  Fast (nanoseconds), simple
  Lost on restart, can't share across servers

Multi-server (Redis):
  Counters in Redis (separate process)
  Shared across all servers
  Survives restarts
  ~0.5ms per check (still fast)

Redis vs In-Memory:
  Both are in-memory storage
  Difference: Redis is a SEPARATE process
  Your app crashes → Redis counters survive
  Multiple servers → all read/write same Redis counters
```

### Industry Scale Progression

```
< 1K users:     In-memory express-rate-limit
1K-10K users:   Add per-user keys, progressive delay
10K-100K:       Redis-backed, sliding window
100K+:          Edge CDN (Cloudflare) + API Gateway + Redis
Netflix scale:  Distributed local+global hybrid
```

### Rate Limit Response Headers (Industry Standard)

```
HTTP/1.1 200 OK
X-RateLimit-Limit: 100          ← your max
X-RateLimit-Remaining: 67       ← how many left
X-RateLimit-Reset: 1773987000   ← when counter resets

HTTP/1.1 429 Too Many Requests
Retry-After: 30                  ← seconds to wait
```

---

## 7. Reverse Proxy vs Rate Limiter vs API Gateway

### The Analogy

```
Reverse Proxy  = Front door receptionist  → "Which server handles this?"
Rate Limiter   = Bouncer                  → "You've had enough, wait."
API Gateway    = Restaurant manager       → Does EVERYTHING
```

### Reverse Proxy

```
Job: Route traffic to the right server
Does: Load balancing, SSL termination, caching, hide servers
Doesn't do: Auth, rate limiting, request transformation
Examples: nginx, HAProxy, Caddy, AWS ALB
```

### Rate Limiter

```
Job: Count requests, block when over limit
Does: Track per-IP/per-user, return 429
Doesn't do: Routing, auth, caching
Lives in: Your app (middleware), reverse proxy, edge CDN, or API gateway
It's a FEATURE, not a service
```

### API Gateway

```
Job: Single entry point for ALL API traffic
Does: EVERYTHING — routing + auth + rate limiting + monitoring +
      request transformation + circuit breaker + API versioning +
      API key management + usage plans
Examples: Kong, AWS API Gateway, Envoy, Traefik

API Gateway = Reverse Proxy + Rate Limiter + Auth + Monitoring + ...
```

### When to Use What

```
1 Express server:              Rate limiter middleware (you are here)
Deploying to Render/Vercel:    They provide reverse proxy (free)
2-5 servers + load balancer:   nginx as reverse proxy + Redis rate limiting
10+ microservices:             API Gateway (Kong / AWS API Gateway)
Massive scale:                 Edge CDN + API Gateway + Service Mesh
```

---

## 8. RBAC — Role-Based Access Control

### What is RBAC?

```
Controls WHO can do WHAT based on their assigned ROLE.

Without RBAC: manage permissions per user (doesn't scale)
With RBAC:    manage permissions per role, assign role to users
```

### Levels of RBAC

#### Level 1: Simple Roles (string on user)

```
User { role: 'admin' }
Check: if (req.user.role === 'admin')
Good for: 2-3 roles, small apps
```

#### Level 2: Permission-Based RBAC

```
Roles map to permissions:
  student: ['quiz:take', 'quiz:view_own', 'topics:read']
  teacher: ['quiz:take', 'quiz:view_own', 'quiz:view_any', 'topics:read']
  admin:   [all permissions]

Check: if (req.user.permissions.includes('quiz:view_any'))

Implementation:
  Config file with role → permissions mapping
  requirePermission() middleware
```

#### Level 3: Resource-Based RBAC

```
Permissions tied to specific resources:
  "Teacher A can see results of Class 10A only"

Check: does teacher's class list include this student's class?
```

#### Level 4: ABAC (Attribute-Based)

```
Rules based on attributes of user + resource + environment:
  "Allow if user.department === resource.department AND time is 9AM-5PM"
Used by: AWS IAM, Google Cloud IAM
```

### Quick Summary: RBAC Levels

```
Level  │ What                     │ Best for
───────┼──────────────────────────┼─────────────────────────────────
  1    │ Role string on user      │ 2-3 roles, small apps
  2    │ Roles → Permissions      │ 5-10 roles, medium apps
  3    │ Resource-level RBAC      │ Multi-tenant, data isolation
  4    │ ABAC (attribute-based)   │ Enterprise, complex policies
```

### Config-Based vs Database-Driven RBAC

```
Config-based (hardcoded):
  // config/permissions.js
  const rolePermissions = {
    student: ['quiz:take', 'quiz:view_own'],
    admin: ['quiz:take', 'quiz:view_any', 'users:manage'],
  };

  ✅ Simple, fast, version controlled
  ❌ Need code deploy to change roles

Database-driven:
  Permission model: { name: 'quiz:take', category: 'quiz' }
  Role model: { name: 'teacher', permissions: [ObjectId, ...] }
  User model: { roleId: ObjectId }

  ✅ Admins create/edit roles from dashboard
  ✅ No code deploy to change permissions
  ❌ More complex (models, admin API, dashboard, caching)
```

### Express.js RBAC Middleware Pattern

```js
// Middleware factory
const requirePermission = (...permissions) => {
  return (req, res, next) => {
    const userPerms = getRolePermissions(req.user.role);
    const hasAll = permissions.every(p => userPerms.includes(p));
    if (!hasAll) return res.status(403).json({ error: 'Forbidden' });
    next();
  };
};

// Usage in routes
router.get('/topics', authMiddleware, requirePermission('topics:read'), getTopics);
router.post('/questions', authMiddleware, requirePermission('questions:create'), createQuestion);
router.delete('/users/:id', authMiddleware, requirePermission('users:manage'), deleteUser);
```

### Caching Permissions for Performance

```
Option A: Cache in JWT payload
  JWT: { userId, role, permissions: ["quiz:take", ...] }
  No DB lookup for permission checks
  Updates on next login / token refresh

Option B: Redis cache
  On login → cache permissions in Redis (TTL: 5 min)
  On request → check Redis first, fallback to DB

Option C: In-memory cache
  Cache role→permissions map in Node.js memory
  Refresh every 5 minutes
```

### How Big Companies Do RBAC

```
GitHub:     read → triage → write → maintain → admin (per repository)
AWS IAM:    JSON policies attached to users/groups/roles
Stripe:     viewer → support → developer → administrator
Slack:      member → admin → owner (per workspace)
```

---

## 9. Role Assignment on Registration

### Approaches

#### Default Role (most common)

```
Everyone registers → gets "student" role
Admin promotes manually

Who does this: Most apps (Twitter, GitHub)
```

#### Self-Selection with Approval

```
Registration form: "I am a: [Student / Teacher]"
Student → active immediately
Teacher → status: "pending" → admin approves

authMiddleware:
  if (req.user.status === 'pending') return 403;
```

#### Invitation-Based

```
Admin → "Invite teacher" → enters email → system sends invite link
Teacher → clicks link → registers with pre-assigned role
Regular registration → always "student"

Who does this: Slack, Notion, Google Workspace
```

#### Auto-Assignment by Domain

```
@school.edu → teacher
@student.school.edu → student
@gmail.com → student (default)
```

### Common Industry Pattern (Combined)

```
Student:         Self-register (default)              → immediate access
Teacher:         Self-register + select role           → pending approval
Content Editor:  Invited by admin only                 → invite link
Admin:           Manually assigned by existing admin   → never self-register
```

---

## 10. Cross-Origin Cookie Issues & Proxy Fix

### The Problem

```
Frontend: https://app.vercel.app
Backend:  https://api.render.com (different domain)

Login → server sets cookie on api.render.com
Next request → browser says "that's a third-party cookie, I won't send it"
→ 401 → instant logout
```

Modern browsers (Chrome 2024+, Safari, Firefox) block third-party cookies even with `SameSite=None; Secure`.

### The Fix: Vercel Proxy

```
Before (broken):
  Browser → api.render.com/api/auth/login
  Cookie domain: render.com (third-party) → BLOCKED by browser

After (fixed):
  Browser → app.vercel.app/api/auth/login
  Vercel rewrites → api.render.com/api/auth/login
  Cookie domain: vercel.app (same-origin) → WORKS

// vercel.json
{
  "rewrites": [{
    "source": "/api/:path*",
    "destination": "https://api.render.com/api/:path*"
  }]
}

// api.ts
const API_BASE_URL = isLocalHost ? 'http://localhost:8000' : '';  // empty = same origin
```

### Why Proxy Works

```
Browser sees: all requests go to vercel.app (same domain)
Vercel silently forwards to render.com (server-to-server, no browser restriction)
Cookie set on vercel.app → same-origin → browser accepts
```

### Other Solutions

```
Custom domain:     app.mysite.com + api.mysite.com (same parent domain)
Authorization header: Store JWT in memory, send as Bearer token (no cookies)
```

---

## 11. HTTP Status Codes — Most Used in APIs

### 2xx — Success

```
200 OK              → Request succeeded. Standard response for GET, PUT, PATCH.
201 Created         → Resource created successfully. Used for POST (register, create question).
204 No Content      → Success but no response body. Used for DELETE.
```

### 3xx — Redirection

```
301 Moved Permanently  → Resource moved to new URL permanently. SEO redirect.
302 Found              → Temporary redirect. Browser follows Location header.
304 Not Modified       → Cached version is still valid. Saves bandwidth.
```

### 4xx — Client Errors

```
400 Bad Request        → Invalid input. Missing fields, wrong format.
                          Example: { email: "" } → "Email is required"

401 Unauthorized       → Not authenticated. No token, expired token, invalid token.
                          Example: API call without cookie → "Access denied"
                          Note: Despite the name, this means "not authenticated", not "not authorized"

403 Forbidden          → Authenticated but not authorized. You ARE logged in, but your role
                          doesn't have permission for this action.
                          Example: Student tries to delete a question → "Forbidden"
                          This is where RBAC kicks in.

404 Not Found          → Resource doesn't exist.
                          Example: GET /api/topics/invalid_id → "Topic not found"

405 Method Not Allowed → Wrong HTTP method. POST to a GET-only endpoint.

409 Conflict           → Request conflicts with current state.
                          Example: Register with existing email → "User already exists"
                          Example: Submit already submitted quiz → "Quiz already submitted"

422 Unprocessable      → Input format is correct but semantically invalid.
                          Example: { "email": "not-an-email" } → "Invalid email format"

423 Locked             → Resource is locked.
                          Example: Account locked after 10 failed logins

429 Too Many Requests  → Rate limit exceeded.
                          Example: 11th login attempt in 15 minutes → "Try again later"
                          Should include: Retry-After header
```

### 5xx — Server Errors

```
500 Internal Server Error → Unhandled server error. Bug in code, DB connection failed.
                            In production: return generic message (don't leak details)

502 Bad Gateway           → Server got invalid response from upstream server.
                            Common with reverse proxies (nginx → your app crashed)

503 Service Unavailable   → Server is overloaded or under maintenance.
                            Should include: Retry-After header

504 Gateway Timeout       → Upstream server didn't respond in time.
                            Common with slow DB queries behind a proxy
```

### 401 vs 403 — The Key Distinction

```
401 Unauthorized (Authentication):
  "Who are you? I don't know you."
  → No token, expired token, invalid token
  → Fix: login again

403 Forbidden (Authorization):
  "I know who you are, but you can't do this."
  → Valid token, but wrong role/permission
  → Fix: get admin to upgrade your role

Flow:
  Request → Is token valid?
              No  → 401 (not authenticated)
              Yes → Does user have permission?
                      No  → 403 (not authorized)
                      Yes → 200 (success)
```

### Quick Reference Table

| Code | Name | General Example |
|------|------|-----------------|
| **200** | **OK** | **User profile fetched successfully** |
| **201** | **Created** | **New blog post created** |
| 204 | No Content | Comment deleted, nothing to return |
| 301 | Moved Permanently | Old URL redirects to new URL |
| 304 | Not Modified | Browser cache is still valid |
| **400** | **Bad Request** | **Missing required field in form** |
| **401** | **Unauthorized** | **JWT expired or not provided** |
| **403** | **Forbidden** | **User tries to access admin panel** |
| **404** | **Not Found** | **GET /api/users/999 → user doesn't exist** |
| 405 | Method Not Allowed | Sending DELETE to a GET-only endpoint |
| **409** | **Conflict** | **Registering with an email that already exists** |
| 422 | Unprocessable Entity | Email field present but format is invalid |
| 423 | Locked | Account locked after too many failed logins |
| **429** | **Too Many Requests** | **Rate limit hit, try again in 30 seconds** |
| **500** | **Internal Server Error** | **Unhandled exception, database crash** |
| 502 | Bad Gateway | Proxy can't reach the backend server |
| 503 | Service Unavailable | Server under maintenance or overloaded |
| 504 | Gateway Timeout | Backend took too long to respond |

---

## Quick Reference: Complete Auth Flow

```
REGISTER
  Client → POST /api/auth/register { name, email, password }
  Server → hash password → create user (role: default) → issue tokens
  Response → set access_token cookie (1h) + refresh_token cookie (7d)
  DB → User created + RefreshToken created

LOGIN
  Client → POST /api/auth/login { email, password }
  Server → validate → delete old refresh tokens → issue new tokens
  Response → set both cookies

AUTHENTICATED REQUEST
  Browser auto-sends cookies → authMiddleware extracts JWT → verifies → attaches user to req
  requirePermission checks user's role permissions → allow or 403

TOKEN EXPIRY (every ~1 hour)
  API returns 401 { code: 'TOKEN_EXPIRED' }
  Frontend apiFetch → calls POST /api/auth/refresh
  Server → validates refresh token from DB → rotates → issues new pair
  Frontend → retries original request → user never notices

LOGOUT
  Client → POST /api/auth/logout
  Server → delete refresh token from DB → clear both cookies
  Frontend → clear localStorage, caches, Zustand store → redirect to /login

REFRESH TOKEN EXPIRY (7 days inactive)
  Refresh attempt → token not in DB or expired → 401 REFRESH_EXPIRED
  Frontend → clear everything → redirect to /login → user must re-login
```