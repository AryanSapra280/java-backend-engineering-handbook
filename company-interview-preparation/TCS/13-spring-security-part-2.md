# 🔐 Spring Security — Part 2

Let's finish the important interview questions and move on.

---

## 25. Access Token vs Refresh Token

### Access Token

Used to access APIs.

```text
Client → API
Authorization: Bearer <access-token>
```

Usually short-lived.

Example:

```text
Access token → 15 minutes
```

### Refresh Token

Used to obtain a new access token when the access token expires.

```text
Refresh token
      ↓
Authorization Server
      ↓
New Access Token
```

Usually longer-lived and should be protected carefully.

### Interview answer

> Access tokens are short-lived credentials used to access protected resources, while refresh tokens are used to obtain new access tokens without requiring the user to authenticate again.

---

# 26. What happens when JWT expires?

Suppose:

```text
Access token
expires after 15 minutes
```

Client makes API request:

```text
JWT expired
     ↓
Authentication fails
     ↓
Usually 401 Unauthorized
```

Client can then use the refresh token:

```text
Refresh Token
     ↓
Authorization Server
     ↓
New Access Token
```

---

# 27. How does Spring Security validate a JWT?

Typically it checks things such as:

```text
1. Token structure
2. Signature
3. Expiration
4. Issuer
5. Audience
6. Required claims
```

The exact checks depend on configuration.

### Important

The server should **not simply decode the JWT and trust its payload**.

It must validate the token's authenticity and relevant claims.

---

# 28. Is JWT encrypted?

**No, not normally.**

A standard signed JWT is generally:

```text
Header
Payload
Signature
```

The header and payload are Base64URL encoded, not encrypted.

Therefore:

> Don't put sensitive information such as passwords or secrets inside a normal JWT payload.

The signature protects integrity/authenticity, not confidentiality.

---

# 29. What is OAuth2?

OAuth 2.0 is an **authorization framework**.

It allows a client to obtain access to protected resources without directly handling the resource owner's credentials.

Example:

```text
User
 ↓
Authorization Server
 ↓
Access Token
 ↓
Client
 ↓
Resource Server
```

Typical components:

```text
Resource Owner
Client
Authorization Server
Resource Server
```

---

# 30. OAuth2 vs JWT

This is a common trap.

They are **not alternatives at the same level**.

### OAuth2

An authorization framework/protocol.

### JWT

A token format.

You can have:

```text
OAuth2
   ↓
Access Token
   ↓
JWT format
```

But OAuth2 access tokens do not have to be JWTs.

---

# 31. What is a Resource Server?

In OAuth2 architecture, the **Resource Server** hosts protected APIs/resources.

Example:

```text
Authorization Server
       ↓
   Access Token
       ↓
Resource Server
       ↓
 /accounts
 /payments
 /transactions
```

Spring Security can configure an application as an OAuth2 Resource Server.

---

# 32. 401 vs 403

Very important.

### 401 Unauthorized

The request does not have valid authentication.

Examples:

```text
No token
Expired token
Invalid token
```

Meaning:

> "I don't know/accept who you are."

### 403 Forbidden

Authentication succeeded, but the user doesn't have sufficient permission.

Example:

```text
USER
 ↓
DELETE /admin/account
 ↓
Authenticated ✓
Authorization ✗
 ↓
403
```

Easy memory:

```text
401 → Authentication problem
403 → Authorization problem
```

---

# 33. Session-based vs JWT authentication

### Session

```text
Login
 ↓
Server creates session
 ↓
Session ID sent to client
 ↓
Server maintains session state
```

### JWT

```text
Login
 ↓
JWT generated
 ↓
Client stores token
 ↓
Token sent with requests
 ↓
Server validates token
```

JWT is commonly used for stateless APIs.

---

# 34. Why use JWT for microservices?

A common interview answer:

> JWT can allow services to validate authentication information without maintaining a centralized HTTP session for every request.

Example:

```text
Client
  ↓ JWT
API Gateway
  ↓
Account Service
Payment Service
Ledger Service
```

Each service can validate the token or rely on a trusted gateway/resource-server architecture depending on the design.

### But don't say:

> "JWT is always better."

The correct choice depends on architecture, token lifecycle, revocation requirements, security requirements, etc.

---

# 35. What happens if we need to immediately revoke a JWT?

This is one disadvantage of purely stateless JWTs.

Suppose:

```text
JWT expires in 1 hour
```

User logs out.

If the server simply validates signature + expiration, the token may remain technically valid until expiry.

Possible approaches:

```text
Short-lived access tokens
+
Refresh-token revocation
```

or maintain a server-side denylist/revocation mechanism when necessary.

### Interview point

> Stateless JWT authentication makes immediate revocation more complicated than server-side sessions.

---

# 36. Why use BCrypt?

Passwords should be stored using a password hashing function rather than plaintext.

BCrypt is designed specifically for password hashing and includes a salt.

Conceptually:

```text
password
   ↓
BCrypt
   ↓
hash
```

Same password does not necessarily produce the same stored hash because of the salt.

During login:

```text
raw password
     ↓
BCrypt verification
     ↓
stored hash
```

---

# 37. What is a Security Filter Chain?

You should be able to explain this clearly.

```text
HTTP Request
     ↓
Security Filter Chain
     ↓
Authentication filters
     ↓
Authorization checks
     ↓
Controller
```

Different filters perform different tasks.

For JWT-based APIs, a JWT authentication mechanism can:

```text
Read Authorization header
        ↓
Extract Bearer token
        ↓
Validate token
        ↓
Create Authentication
        ↓
SecurityContext
```

---

# 38. What is `SecurityContextHolder`?

It provides access to the current security context.

Example:

```java
Authentication auth =
    SecurityContextHolder
        .getContext()
        .getAuthentication();
```

Then:

```java
String username = auth.getName();
```

or:

```java
Collection<? extends GrantedAuthority>
    authorities = auth.getAuthorities();
```

---

# 39. How would you secure an endpoint for ADMIN?

```java
@Bean
SecurityFilterChain securityFilterChain(
        HttpSecurity http) throws Exception {

    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/admin/**")
        .hasRole("ADMIN")
        .anyRequest()
        .authenticated()
    );

    return http.build();
}
```

Or method-level:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteAccount(Long id) {
}
```

---

# 40. What is stateless security configuration?

For a JWT-based API, you commonly configure:

```java
http
    .sessionManagement(session ->
        session.sessionCreationPolicy(
            SessionCreationPolicy.STATELESS
        )
    );
```

Meaning Spring Security should not create/use an HTTP session to maintain authentication state in the traditional way.

---

# 41. Why disable CSRF for some REST APIs?

For a typical stateless API where authentication is supplied explicitly through:

```http
Authorization: Bearer <token>
```

CSRF protection may not be needed in the same way as cookie-based authentication.

But don't answer:

> "REST APIs don't need CSRF."

That's too broad.

Better:

> "For stateless APIs using bearer tokens that are not automatically attached by the browser, CSRF risk is different from cookie-based authentication, so CSRF protection may be disabled depending on the architecture."

---

# 42. What is CORS configuration in Spring Security?

Example:

```java
http.cors(...)
```

Then configure allowed origins, methods, headers, etc.

Conceptually:

```text
Frontend
https://app.example.com

API
https://api.example.com
```

The browser sees different origins.

Server must allow the required cross-origin requests.

---

# 43. What is authentication manager?

`AuthenticationManager` is responsible for processing authentication requests.

Conceptually:

```text
Authentication request
       ↓
AuthenticationManager
       ↓
AuthenticationProvider
       ↓
UserDetailsService / credentials validation
       ↓
Authenticated object
```

---

# 44. What is `AuthenticationProvider`?

It performs a particular type of authentication.

For username/password authentication, an authentication provider can use:

```text
UserDetailsService
+
PasswordEncoder
```

Conceptually:

```text
AuthenticationManager
        ↓
AuthenticationProvider
        ↓
UserDetailsService
        ↓
Database
```

---

# 45. AuthenticationManager vs AuthenticationProvider

Easy distinction:

```text
AuthenticationManager
    ↓
coordinates authentication

AuthenticationProvider
    ↓
performs a specific authentication mechanism
```

You can have multiple providers.

Example:

```text
AuthenticationManager
       ↓
 ┌───────────────┐
 ↓               ↓
Provider A     Provider B
LDAP           DB/password
```

---

# 🔥 Spring Security rapid-fire

| Question                 | Answer                                                  |
| ------------------------ | ------------------------------------------------------- |
| Authentication?          | Verify identity                                         |
| Authorization?           | Check permissions                                       |
| SecurityFilterChain?     | Security filters applied to requests                    |
| SecurityContext?         | Holds current authentication                            |
| UserDetails?             | Security representation of user                         |
| UserDetailsService?      | Loads user information                                  |
| PasswordEncoder?         | Password hashing/verification                           |
| JWT?                     | Token format                                            |
| OAuth2?                  | Authorization framework                                 |
| Access token?            | Used to access protected resources                      |
| Refresh token?           | Used to obtain new access token                         |
| 401?                     | Authentication failure/missing/invalid credentials      |
| 403?                     | Authenticated but insufficient permission               |
| `hasRole("ADMIN")`?      | Typically checks `ROLE_ADMIN`                           |
| `hasAuthority("ADMIN")`? | Checks `ADMIN`                                          |
| CSRF?                    | Protection against forged authenticated requests        |
| CORS?                    | Browser cross-origin access control                     |
| Stateless?               | No traditional server-side session authentication state |
| BCrypt?                  | Password hashing algorithm                              |
| SecurityContextHolder?   | Access current security context                         |
| AuthenticationManager?   | Coordinates authentication                              |
| AuthenticationProvider?  | Performs a specific authentication mechanism            |

---

# ✅ Spring Security DONE

Next:

# 👉 Spring AOP

We'll cover:

* What is AOP?
* Aspect / Advice / Pointcut / Join Point
* `@Before`
* `@After`
* `@AfterReturning`
* `@AfterThrowing`
* `@Around`
* `@Pointcut`
* Spring AOP proxy
* JDK vs CGLIB proxy
* self-invocation trap
* `@Transactional` connection to AOP
* real-world logging/security/performance examples
* common interview traps

Then **Spring Batch → SQL/Postgres → Microservices → Kafka → Redis → Docker → System Design**.
