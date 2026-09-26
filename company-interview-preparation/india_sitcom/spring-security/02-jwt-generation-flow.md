# 🔐 Spring Security — Part 2: Authentication Architecture

Now we get into the **core Spring Security machinery**.

If you understand this flow, JWT becomes much easier:

```text
Authentication
      ↓
AuthenticationManager
      ↓
AuthenticationProvider
      ↓
UserDetailsService
      ↓
PasswordEncoder
```

Don't memorize these as isolated interfaces. Understand **who calls whom and why**.

---

# 1. First: What happens when a user logs in?

Suppose our payment platform has:

```text
username = merchant123
password = secret
```

Client sends:

```http
POST /auth/login
Content-Type: application/json

{
  "username": "merchant123",
  "password": "secret"
}
```

Conceptually:

```text
Client
   |
   | username + password
   ↓
Authentication
   |
   ↓
AuthenticationManager
   |
   ↓
AuthenticationProvider
   |
   ↓
UserDetailsService
   |
   ↓
Database
```

Then:

```text
Database
   ↓
stored password hash
   ↓
PasswordEncoder.matches()
   ↓
password correct?
   |
   ├── NO  → authentication failure
   |
   └── YES
          ↓
     authenticated
          ↓
     generate JWT
```

That's the basic architecture.

---

# 2. `Authentication`

Earlier we said:

> `Authentication` represents the identity and authentication information.

But during login, there's an interesting detail.

Before authentication:

```java
UsernamePasswordAuthenticationToken
```

can contain:

```text
principal   = merchant123
credentials = secret
authenticated = false
```

Think:

```text
"Here are the credentials.
Please authenticate me."
```

After successful authentication, Spring has an authenticated `Authentication` containing the principal and authorities.

So conceptually:

```text
BEFORE

Authentication
 ├── username
 ├── password
 └── authenticated = false


AFTER

Authentication
 ├── principal = merchant123
 ├── authorities = ROLE_MERCHANT
 └── authenticated = true
```

---

# 3. `AuthenticationManager`

Now we have:

```java
AuthenticationManager
```

Its job is basically:

> **Take authentication credentials and try to authenticate them.**

The key method is:

```java
Authentication authenticate(Authentication authentication)
```

So your login flow conceptually becomes:

```java
Authentication authentication =
        authenticationManager.authenticate(
            usernamePasswordAuthenticationToken
        );
```

The `AuthenticationManager` doesn't necessarily know how to authenticate every type of credential itself.

Instead, it delegates.

---

# 4. `AuthenticationProvider`

This is where things become interesting.

`AuthenticationManager` delegates authentication to an appropriate:

```java
AuthenticationProvider
```

For username/password authentication, one commonly used provider is:

```text
DaoAuthenticationProvider
```

So:

```text
AuthenticationManager
        |
        ↓
DaoAuthenticationProvider
```

Think:

> AuthenticationManager = **dispatcher**

> AuthenticationProvider = **actual authentication specialist**

---

# 5. `DaoAuthenticationProvider`

The name looks complicated, but the concept is simple.

DAO = Data Access Object.

`DaoAuthenticationProvider` commonly authenticates username/password using:

```text
UserDetailsService
+
PasswordEncoder
```

So:

```text
DaoAuthenticationProvider
       |
       ├── UserDetailsService
       |
       └── PasswordEncoder
```

This is a **very important diagram**.

---

# 6. `UserDetailsService`

Now suppose the user sends:

```text
username = merchant123
```

Spring needs to find this user.

That's where:

```java
UserDetailsService
```

comes in.

It has:

```java
UserDetails loadUserByUsername(String username)
```

Conceptually:

```java
@Service
public class CustomUserDetailsService
        implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String username) {

        User user = userRepository
                .findByUsername(username)
                .orElseThrow(
                    () -> new UsernameNotFoundException(username)
                );

        return User.withUsername(user.getUsername())
                .password(user.getPassword())
                .roles("MERCHANT")
                .build();
    }
}
```

So:

```text
username
   ↓
UserDetailsService
   ↓
Database
   ↓
User
   ↓
UserDetails
```

---

# 7. What is `UserDetails`?

`UserDetails` is Spring Security's representation of the user needed for authentication/authorization.

Conceptually:

```text
UserDetails
 ├── username
 ├── password
 ├── authorities
 ├── accountNonExpired
 ├── accountNonLocked
 ├── credentialsNonExpired
 └── enabled
```

For example:

```text
username:
merchant123

password:
$2a$10$....

authorities:
ROLE_MERCHANT
PAYMENT_READ
PAYMENT_CREATE
```

Your actual database entity does **not have to be exactly the same class** as `UserDetails`.

You can map:

```text
User DB Entity
      ↓
UserDetails
```

---

# 8. PasswordEncoder 🔥

This is extremely important.

You should **never store**:

```text
password = "secret123"
```

in your database.

Instead:

```text
secret123
    ↓
PasswordEncoder
    ↓
hash
    ↓
$2a$10$....
```

A common implementation is:

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

Database:

```text
username: merchant123
password: $2a$10$7....
```

When user logs in:

```text
User enters:

secret123

       ↓

PasswordEncoder.matches(
    "secret123",
    "$2a$10$7..."
)

       ↓

true
```

---

# 9. Very important: Does Spring decrypt the password?

**No.**

This is a classic interview question.

Password hashing is designed to be one-way.

You don't do:

```text
storedHash
    ↓
decrypt
    ↓
password
```

Instead:

```text
raw password
     ↓
hashing algorithm
     ↓
compare against stored hash
```

So the interviewer may ask:

> "How does Spring Security validate a password if it doesn't decrypt the stored password?"

Answer:

> "It uses the PasswordEncoder to verify the raw password against the stored encoded password, typically using a method such as `matches()`. The stored password isn't decrypted."

---

# 10. Complete username/password flow

Now put everything together.

Client:

```http
POST /login
```

with:

```json
{
  "username": "merchant123",
  "password": "secret"
}
```

Then:

```text
                  LOGIN REQUEST
                       |
                       ↓
        UsernamePasswordAuthenticationToken
                       |
                       ↓
              AuthenticationManager
                       |
                       ↓
             DaoAuthenticationProvider
                       |
             ┌─────────┴─────────┐
             ↓                   ↓
    UserDetailsService      PasswordEncoder
             |                   |
             ↓                   |
        Database                 |
             |                   |
             └───────┬───────────┘
                     ↓
                Authentication
                     |
                     ↓
                SUCCESS
                     |
                     ↓
               Generate JWT
                     |
                     ↓
                 Client
```

🔥 **This flow is worth knowing extremely well.**

---

# 11. Where does JWT come into this?

This is where many people confuse two different things.

### Username/password authentication

Used to establish identity:

```text
username + password
       ↓
authenticate
       ↓
JWT issued
```

### Subsequent API requests

The client doesn't normally send the password again.

Instead:

```http
Authorization: Bearer eyJ...
```

Then:

```text
JWT
 ↓
validate
 ↓
Authentication
 ↓
SecurityContext
 ↓
Authorization
 ↓
Controller
```

So:

```text
LOGIN

username/password
       ↓
AuthenticationManager
       ↓
JWT generated


SUBSEQUENT REQUEST

JWT
 ↓
Spring Security
 ↓
validate
 ↓
SecurityContext
 ↓
Controller
```

That's a **very important distinction**.

---

# 12. Does `UserDetailsService` get called on every JWT request?

Generally, **not necessarily**.

This is another interview trap.

With a stateless JWT resource-server setup:

```text
Request
   |
   ↓
JWT
   |
   ↓
JwtDecoder
   |
   ↓
validate JWT
   |
   ↓
Authentication
```

The service doesn't have to query your user database on every request just to validate the JWT.

For example:

```text
GET /payments

Authorization: Bearer <JWT>
```

Spring can validate:

```text
signature
issuer
expiration
audience (if configured)
```

and derive authorities from the JWT.

So:

```text
JWT request
    ↓
JWT validation
    ↓
SecurityContext
```

rather than:

```text
JWT request
    ↓
database
    ↓
find user
    ↓
check password
```

This is one of the benefits of stateless token-based authentication.

---

# 13. But then how does Spring know the user's role?

The JWT can contain claims.

For example:

```json
{
  "sub": "merchant123",
  "roles": [
    "MERCHANT"
  ]
}
```

Spring can convert claims into authorities.

Then:

```java
.hasRole("MERCHANT")
```

can be used for authorization.

We'll implement this properly when we reach **JWT + RBAC**.

---

# 14. Authentication vs Authorization — now with the architecture

This distinction should now be crystal clear.

### Authentication

```text
Who are you?

username/password
       ↓
AuthenticationManager
       ↓
AuthenticationProvider
       ↓
authenticated identity
```

### Authorization

```text
What can you do?

Authentication
       ↓
Authorities
       ↓
Authorization rules
       ↓
allow / deny
```

For example:

```java
.requestMatchers("/admin/**")
.hasRole("ADMIN")
```

means:

```text
Is user authenticated?
        |
        ↓
What authorities do they have?
        |
        ↓
ROLE_ADMIN?
        |
     ┌──┴──┐
    YES    NO
     |      |
    200    403
```

---

# 15. `401` vs `403` 🔥

This is **very commonly asked**.

### `401 Unauthorized`

Usually means:

> The request doesn't have valid authentication.

Examples:

```text
No JWT
Expired JWT
Invalid JWT
Malformed JWT
Invalid credentials
```

Think:

> **"I don't know/accept who you are."**

---

### `403 Forbidden`

Authentication exists, but:

> You don't have permission.

Example:

```text
JWT valid

User:
ROLE_MERCHANT

Endpoint:
requires ROLE_ADMIN
```

Result:

```http
403 Forbidden
```

Think:

> **"I know who you are, but you're not allowed to do this."**

Memorize:

```text
401 → Authentication problem
403 → Authorization problem
```

---

# 16. Now let's connect this to our Payment Service

Imagine:

```text
POST /api/v1/payment
```

requires authentication.

But:

```text
POST /auth/login
```

must be public.

Configuration conceptually:

```java
http
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/auth/**").permitAll()
        .requestMatchers("/api/v1/payment/**")
            .authenticated()
        .anyRequest()
            .authenticated()
    );
```

So:

```text
/auth/login
    ↓
PUBLIC


/api/v1/payment
    ↓
JWT REQUIRED
```

Then later:

```java
.requestMatchers(HttpMethod.POST, "/api/v1/payment")
    .hasAuthority("PAYMENT_CREATE")
```

Now we have:

```text
JWT
 ↓
Authentication
 ↓
PAYMENT_CREATE
 ↓
POST /payment
```

---

# 17. One critical distinction: login implementation vs Resource Server

For your interview, understand that these are related but distinct responsibilities.

### Authentication server / login

Potentially:

```text
username/password
       ↓
authenticate
       ↓
issue JWT
```

This could be handled by your own authentication service or by something like Keycloak/another authorization server.

### Resource server

Your payment microservice:

```text
JWT
 ↓
validate JWT
 ↓
authorize request
 ↓
payment API
```

In a real microservice architecture, you will often have:

```text
                 Keycloak
             Authorization Server
                    |
                    | JWT
                    ↓
Client ─────────→ API Gateway
                    |
                    ↓
             Payment Service
             Resource Server
                    |
                    ↓
                 MongoDB
```

Your payment service doesn't need to implement a full identity provider just to validate JWTs.

---

# 18. The interview question I expect

They might say:

> **"You have a Spring Boot payment microservice. Implement JWT authentication."**

Your thought process should be:

### Step 1

Add resource-server/JWT dependencies.

### Step 2

Configure:

```java
SecurityFilterChain
```

### Step 3

Configure JWT validation:

```text
issuer-uri
```

or a public key / `JwtDecoder`, depending on the architecture.

### Step 4

Configure:

```java
.anyRequest().authenticated()
```

### Step 5

Configure public endpoints:

```java
.requestMatchers("/auth/**").permitAll()
```

### Step 6

Configure authorization:

```java
.hasRole(...)
.hasAuthority(...)
```

### Step 7

Test:

```http
Authorization: Bearer <token>
```

### Step 8

Handle:

```text
401
403
expired JWT
invalid JWT
insufficient authority
```

That is the implementation mindset.

---

# 🧠 What you should know after this section

You should now have this hierarchy in your notes:

```text
Spring Security
│
├── SecurityFilterChain
│
├── Authentication
│
├── AuthenticationManager
│
├── AuthenticationProvider
│      │
│      └── DaoAuthenticationProvider
│
├── UserDetailsService
│
├── UserDetails
│
├── PasswordEncoder
│
├── SecurityContext
│
├── SecurityContextHolder
│
└── GrantedAuthority
```

And the username/password flow:

```text
Client
  |
  | username + password
  ↓
UsernamePasswordAuthenticationToken
  |
  ↓
AuthenticationManager
  |
  ↓
AuthenticationProvider
  |
  ↓
UserDetailsService → DB
  |
  ↓
PasswordEncoder
  |
  ↓
Authentication SUCCESS
  |
  ↓
JWT generated
  |
  ↓
Client
```

Then subsequent requests:

```text
Client
  |
  | Bearer JWT
  ↓
Security Filter Chain
  |
  ↓
JWT authentication
  |
  ↓
JwtDecoder
  |
  ↓
Authentication
  |
  ↓
SecurityContext
  |
  ↓
Authorization
  |
  ↓
Controller
```

---

## 🔥 Next: Part 3 — JWT from zero, then we code it

Next we'll go **much more practical**.

We'll take one actual JWT and dissect:

```text
HEADER.PAYLOAD.SIGNATURE
```

Then I'll explain:

* `HS256` vs `RS256`
* private key vs public key
* `issuer`
* `audience`
* `subject`
* `expiration`
* `scope`
* roles/authorities
* how Spring's `JwtDecoder` works
* `BearerTokenAuthenticationFilter`
* `JwtAuthenticationProvider`
* why the DB isn't queried on every request
* how Keycloak fits into this
* and then **we'll actually implement JWT authentication in your Payment Service** rather than just discussing it.

That implementation is the part I especially want you to do, because after we write it once, I'll start asking you interview-style questions against **your own code**.


# OAuth2 is a protocol/framework that defines the standard flows for obtaining and using an access token. The client sends the required OAuth2 request, along with the required credentials such as client_id, client_secret, and the grant type, to the Authorization Server. The Authorization Server validates the request and, if everything is valid, issues an access token. In our case, that access token is in JWT format. The client then sends this JWT as a Bearer token in the Authorization header whenever it calls a protected API. The Resource Server receives the JWT, validates it, identifies the client/user and its authorities, and then allows or denies access to the API.