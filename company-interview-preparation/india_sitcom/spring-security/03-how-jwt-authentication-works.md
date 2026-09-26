Spring Security works primarily through a security filter chain. When a request comes in with a Bearer JWT, the BearerTokenAuthenticationFilter extracts the token and initiates authentication. The authentication is delegated through the AuthenticationManager to an appropriate AuthenticationProvider. For JWT resource-server authentication, this is typically the JwtAuthenticationProvider, which uses a JwtDecoder to validate the JWT signature and claims. If validation succeeds, Spring creates an authenticated Authentication object and stores it in the SecurityContext. The authorization layer then checks the authorities or roles associated with that authentication. If authorization succeeds, the request proceeds to the controller; otherwise Spring returns an appropriate error such as 401 or 403


Exactly! **Your Spring Boot application is the Resource Server** when it receives a JWT and validates it.

The easiest way to understand the whole thing is to separate it into **two phases**:

1. **Getting the token** — Authorization Server / Keycloak side
2. **Using and validating the token** — Your Resource Server side

---

# 🔐 Complete OAuth2 + JWT + Spring Security Flow

Let's assume:

* **User** → Aryan
* **Keycloak** → Authorization Server
* **Payment Service** → Your Spring Boot Resource Server
* **JWT** → Access token

```text
                    PHASE 1: GET TOKEN
                    =================

       User
        |
        | username + password
        ↓
   ┌─────────────┐
   │  Keycloak   │
   │ Authorization│
   │    Server   │
   └──────┬──────┘
          |
          | 1. Authenticate user
          | 2. Check user exists
          | 3. Check password
          | 4. Check client
          | 5. Create JWT
          ↓
      Access Token
       (JWT)
          |
          ↓
       Client
```

Then:

```text
                    PHASE 2: CALL API
                    =================

Client
  |
  | Authorization: Bearer <JWT>
  ↓
┌──────────────────────────────┐
│  Your Payment Service        │
│  Spring Boot                 │
│  RESOURCE SERVER             │
│                              │
│  Security Filter Chain       │
│          ↓                   │
│  BearerTokenAuthentication   │
│  Filter                      │
│          ↓                   │
│  AuthenticationManager       │
│          ↓                   │
│  JwtAuthenticationProvider   │
│          ↓                   │
│  JwtDecoder                  │
│          ↓                   │
│  Validate JWT                 │
│          ↓                   │
│  SecurityContext              │
│          ↓                   │
│  Authorization               │
│          ↓                   │
│       Controller             │
└──────────────────────────────┘
```

Now let's understand **where the public key comes from**.

---

# 1. Keycloak creates the JWT

Suppose the user logs in.

```text
User
 |
 | username/password
 ↓
Keycloak
```

Keycloak verifies:

```text
Does user exist?
Is password correct?
Is account enabled?
Does this client have permission?
```

If everything is valid, Keycloak creates a JWT.

Conceptually:

```text
JWT =
HEADER.PAYLOAD.SIGNATURE
```

For example:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "abc123"
}
```

Payload:

```json
{
  "sub": "user123",
  "iss": "https://keycloak.example.com/realms/payment",
  "aud": "payment-service",
  "exp": 1790000000,
  "scope": "payment:read payment:write"
}
```

Keycloak signs this JWT using its **private key**.

```text
             Keycloak
                 
       Private Key
            |
            ↓
       Sign JWT
            |
            ↓
         JWT
```

The important point:

> **The private key never needs to be given to your Payment Service.**

---

# 2. So where does my application get the public key?

This is the part you were asking about. 👇

Keycloak exposes its **public keys** through a **JWKS endpoint**.

JWKS = **JSON Web Key Set**

Conceptually:

```text
Keycloak

Private Key
    |
    | signs JWT
    ↓

JWT

Public Key
    |
    | exposed through JWKS endpoint
    ↓

https://keycloak.../protocol/openid-connect/certs
```

Your Resource Server can obtain that public key from Keycloak.

---

# 3. How does Spring Boot know where the keys are?

Usually you configure:

```properties
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://keycloak.example.com/realms/payment
```

You don't normally hard-code the public key yourself.

Spring Security uses the issuer information to discover the Authorization Server's metadata and the JWKS endpoint.

Conceptually:

```text
Your application
      |
      | issuer-uri
      ↓
Keycloak issuer
      |
      | metadata discovery
      ↓
JWKS endpoint
      |
      ↓
Public Key
```

Spring Security then creates/configures a `JwtDecoder`.

---

# 4. What is JWKS?

Think of it as:

> **"Here are the public keys that can be used to verify tokens that I signed."**

For example:

```json
{
  "keys": [
    {
      "kid": "abc123",
      "kty": "RSA",
      "alg": "RS256",
      "use": "sig"
    }
  ]
}
```

The important field is:

```text
kid
```

The JWT header contains:

```json
{
    "alg": "RS256",
    "kid": "abc123"
}
```

Spring Security sees:

```text
kid = abc123
```

and selects the corresponding public key from the JWKS.

---

# 5. Now the user calls your Payment API

The client sends:

```http
POST /api/v1/payment
Authorization: Bearer eyJhbGciOiJSUzI1Ni...
```

Now your application takes over.

---

# 6. Request enters Spring Security Filter Chain

Before your request reaches:

```java
@PostMapping
public PaymentResponse createPayment(...) {
}
```

it first passes through Spring Security.

```text
HTTP Request
     |
     ↓
Spring Security Filter Chain
```

One important filter is:

```text
BearerTokenAuthenticationFilter
```

It looks for:

```http
Authorization: Bearer <JWT>
```

and extracts:

```text
JWT
```

---

# 7. AuthenticationManager gets involved

The filter passes the token into Spring Security's authentication machinery.

Conceptually:

```text
BearerTokenAuthenticationFilter
             |
             ↓
   AuthenticationManager
             |
             ↓
   JwtAuthenticationProvider
```

For JWT authentication, the relevant provider is:

```text
JwtAuthenticationProvider
```

---

# 8. JwtAuthenticationProvider uses JwtDecoder

Now:

```text
JwtAuthenticationProvider
          |
          ↓
     JwtDecoder
```

The `JwtDecoder` validates the JWT.

There are several things to validate.

### A. Signature

This is the big one.

Keycloak did:

```text
Private Key
    +
 JWT
    ↓
Signature
```

Your Resource Server does:

```text
JWT + Public Key
      ↓
Verify Signature
```

If the signature is invalid:

```text
401 Unauthorized
```

---

# 9. Claims are also validated

Spring Security can validate things like:

### Issuer

```json
"iss": "https://keycloak.example.com/realms/payment"
```

Does this token actually come from the expected Authorization Server?

### Expiration

```json
"exp": 1790000000
```

Has the token expired?

### Audience

```json
"aud": "payment-service"
```

Is this token intended for my service?

Depending on configuration, additional claims/requirements can be validated too.

---

# 10. If JWT is valid → Authentication object is created

Suppose everything passes.

```text
JWT
 ↓
Signature valid
 ↓
Issuer valid
 ↓
Expiration valid
 ↓
Claims valid
 ↓
Authenticated
```

Spring Security creates an `Authentication` object, commonly:

```text
JwtAuthenticationToken
```

This contains information derived from the JWT.

For example:

```text
Principal:
    user123

Authorities:
    PAYMENT_READ
    PAYMENT_WRITE
```

---

# 11. Authentication goes into SecurityContext

Spring Security stores the authenticated identity in:

```text
SecurityContext
```

and commonly accesses it through:

```java
SecurityContextHolder
```

Conceptually:

```text
JWT
 ↓
Authentication
 ↓
SecurityContext
```

Now Spring Security knows:

> "This request is from user123 and they have these authorities."

---

# 12. Authorization happens

Now Spring Security checks:

```java
.requestMatchers("/api/v1/payment/**")
.hasAuthority("PAYMENT_WRITE")
```

Suppose the JWT contains:

```text
PAYMENT_WRITE
```

Then:

```text
Authorization = SUCCESS
```

Request continues.

If the user is authenticated but doesn't have the required authority:

```text
403 Forbidden
```

---

# 13. Finally your Controller executes

Only after authentication + authorization succeed:

```text
Controller
```

gets called.

For example:

```java
@PostMapping
public PaymentResponse createPayment(
        @Valid @RequestBody CreatePaymentRequest request) {

    return paymentService.createPayment(request);
}
```

And you can access the authenticated user:

```java
Authentication authentication =
        SecurityContextHolder
                .getContext()
                .getAuthentication();
```

---

# 🔥 Full Flow You Should Memorize

For your interview, remember this exact flow:

```text
                    TOKEN CREATION
                    ==============

User
 |
 | username/password
 ↓
Authorization Server / Keycloak
 |
 | authenticate user
 | validate credentials
 | create JWT
 | sign JWT using PRIVATE KEY
 ↓
JWT Access Token
 |
 ↓
Client
```

Then:

```text
                    API REQUEST
                    ============

Client
 |
 | Authorization: Bearer <JWT>
 ↓
Payment Service
(Resource Server)
 |
 ↓
Security Filter Chain
 |
 ↓
BearerTokenAuthenticationFilter
 |
 | extracts JWT
 ↓
AuthenticationManager
 |
 ↓
JwtAuthenticationProvider
 |
 ↓
JwtDecoder
 |
 | gets/uses public key
 | from Authorization Server JWKS
 ↓
Validate JWT
 |
 ├── Signature
 ├── Issuer
 ├── Expiration
 ├── Audience
 └── other configured claims
 |
 ↓
JwtAuthenticationToken
 |
 ↓
SecurityContext
 |
 ↓
Authorization
 |
 ├── Not authenticated → 401
 ├── Authenticated but insufficient authority → 403
 |
 ↓
Controller
 |
 ↓
Service
 |
 ↓
Database / other services
```

---

# ⭐ The Keycloak/Public-Key Part

This is the sentence I want you to remember for the interview:

> **"In a JWT-based OAuth2 resource-server setup, the Authorization Server signs the JWT using its private key and exposes the corresponding public keys through a JWKS endpoint. My Spring Boot Resource Server is configured with the issuer URI, and Spring Security uses that information to discover the authorization server's keys and configure the JwtDecoder. When a request arrives, the JwtDecoder uses the appropriate public key to verify the JWT signature and validates claims such as issuer and expiration."**

And then if the interviewer asks:

### "Does your application call Keycloak for every request?"

You say:

> **"Not necessarily. With JWT-based validation, the Resource Server can validate the token locally using the public key. Spring Security can cache the discovered signing keys, so it doesn't need to call Keycloak for every request."**

That's a **very important distinction**.

```text
JWT approach:

Request → Your Service → Local JWT validation
                         ↑
                    Public Key
```

versus an opaque-token/introspection approach:

```text
Request → Your Service → Authorization Server
                              |
                         "Is token valid?"
```

For your current interview prep, **this JWT flow is the one you should be able to draw on a whiteboard from memory.**


Yes — **this is enough for the interview**, especially given your limited prep time. 👍

You should be able to confidently explain these **7 things**:

1. **Authentication vs Authorization**
2. **OAuth2 vs JWT vs Keycloak**
3. **Authorization Server vs Resource Server**
4. **How Keycloak issues and signs a JWT**
5. **How your application gets the public key via JWKS**
6. **Spring Security request flow**

   ```text
   Request
   → Security Filter Chain
   → BearerTokenAuthenticationFilter
   → AuthenticationManager
   → JwtAuthenticationProvider
   → JwtDecoder
   → SecurityContext
   → Authorization
   → Controller
   ```
7. **401 vs 403**

And especially remember this:

> **Keycloak authenticates the user and issues the JWT. My Resource Server doesn't normally ask Keycloak to validate the JWT on every request. It validates the JWT locally using the public signing key obtained through JWKS.**

If they go one level deeper, you know what to say about:

* `issuer-uri`
* `JwtDecoder`
* `AuthenticationManager`
* `AuthenticationProvider`
* `SecurityContext`
* `RS256`
* public/private keys
* JWKS
* token expiration

### What I would NOT spend time on right now

Don't dive into:

* OAuth2 internals/spec details
* custom security filters
* writing your own JWT decoder
* Keycloak administration
* custom `JwtAuthenticationConverter` unless they ask
* OAuth2 authorization-code flow in exhaustive detail
* cryptography mathematics

You've got the **interview-relevant mental model** now.

**I'd move on.** Next high-value topic should be **JPA + SQL**, because that's likely to generate considerably more practical coding and follow-up questions for this role.
