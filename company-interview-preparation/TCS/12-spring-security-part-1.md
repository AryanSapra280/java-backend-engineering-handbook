# 🔐 Spring Security — Part 1

Let's keep this **interview-speed**, but enough to answer follow-ups confidently.

---

## 1. What is Spring Security?

**Interview answer:**

> Spring Security is a framework for securing Spring applications. It provides authentication, authorization, protection against common attacks, and integration with mechanisms such as session-based authentication, JWT, OAuth2, etc.

Think:

```text
Authentication → Who are you?
Authorization  → What are you allowed to do?
```

Example:

```text
User logs in
   ↓
Authentication
   ↓
JWT issued
   ↓
Request comes with JWT
   ↓
Authorization
   ↓
Can this user access /admin?
```

---

# 2. Authentication vs Authorization

### Authentication

Verifies identity.

```text
"Who are you?"
```

Example:

```text
username = aryan
password = ****
```

System verifies credentials.

### Authorization

Determines permissions.

```text
"What are you allowed to do?"
```

Example:

```text
ADMIN → DELETE account
USER  → VIEW account
```

### Easy interview line

> Authentication establishes identity; authorization determines what that authenticated identity can access.

---

# 3. How does Spring Security work?

The most important concept:

# Security Filter Chain

A request roughly flows like:

```text
Client
  ↓
Security Filters
  ↓
Authentication
  ↓
Authorization
  ↓
Controller
```

Spring Security uses a chain of servlet filters to intercept requests **before they reach your controller**.

---

# 4. What is `SecurityFilterChain`?

Modern Spring Security configuration commonly looks like:

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/public/**").permitAll()
            .requestMatchers("/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated()
        );

    return http.build();
}
```

This defines how HTTP requests should be secured.

### Example

```text
/public/** → anyone
/admin/**  → ADMIN
everything else → authenticated
```

---

# 5. What is a Security Filter?

A security filter intercepts HTTP requests and performs security-related processing.

For example, a JWT filter can:

```text
Request
  ↓
Read Authorization header
  ↓
Extract JWT
  ↓
Validate JWT
  ↓
Extract user information
  ↓
Create Authentication
  ↓
Put it into SecurityContext
  ↓
Continue request
```

---

# 6. What is `SecurityContext`?

Spring Security stores the currently authenticated user's information in:

```java
SecurityContext
```

You can retrieve authentication:

```java
Authentication authentication =
    SecurityContextHolder
        .getContext()
        .getAuthentication();
```

Then:

```java
authentication.getName();
authentication.getAuthorities();
```

Conceptually:

```text
SecurityContext
      ↓
Authentication
      ↓
Principal + Authorities
```

---

# 7. What is `Authentication`?

`Authentication` represents the current authentication information.

It can contain:

```text
Principal
Credentials
Authorities
Authenticated status
```

Example:

```java
Authentication auth =
    SecurityContextHolder
        .getContext()
        .getAuthentication();
```

Then:

```java
auth.getName();
auth.getAuthorities();
```

---

# 8. What is `UserDetails`?

`UserDetails` represents user information understood by Spring Security.

Example:

```java
public class CustomUserDetails
        implements UserDetails {

    private String username;
    private String password;

    private Collection<? extends GrantedAuthority>
        authorities;

    // methods...
}
```

It provides things like:

```text
username
password
authorities
account status
account expiration
credentials expiration
```

---

# 9. What is `UserDetailsService`?

`UserDetailsService` loads a user from your user store.

Interface:

```java
public interface UserDetailsService {

    UserDetails loadUserByUsername(String username);
}
```

Typical implementation:

```java
@Service
public class CustomUserDetailsService
        implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String username) {

        User user = userRepository
            .findByUsername(username)
            .orElseThrow();

        return new CustomUserDetails(user);
    }
}
```

Flow:

```text
Login request
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

# 10. What is `PasswordEncoder`?

**Never store raw passwords.**

Spring Security provides:

```java
PasswordEncoder
```

Common choice:

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

When registering:

```java
String encoded =
    passwordEncoder.encode(password);
```

Store the encoded value.

During authentication:

```text
Raw password
     ↓
PasswordEncoder
     ↓
Compare against stored hash
```

### Important trap

BCrypt is a **password hashing algorithm**, not encryption.

You don't decrypt the stored password.

---

# 11. What is JWT?

JWT = **JSON Web Token**.

It is commonly used for stateless authentication.

Typical structure:

```text
header.payload.signature
```

Example conceptually:

```text
xxxxx.yyyyy.zzzzz
```

Payload may contain:

```json
{
  "sub": "user123",
  "roles": ["USER"],
  "exp": 1790000000
}
```

The server verifies the signature before trusting the claims.

---

# 12. JWT authentication flow

This is VERY important for interviews.

### Login

```text
Client
  ↓
POST /login
username + password
  ↓
Authentication
  ↓
Database
  ↓
Credentials verified
  ↓
JWT generated
  ↓
Client
```

Then subsequent request:

```text
GET /accounts
Authorization: Bearer <JWT>
```

Flow:

```text
Client
  ↓
JWT
  ↓
Security Filter
  ↓
Validate JWT
  ↓
Extract user/authorities
  ↓
SecurityContext
  ↓
Authorization
  ↓
Controller
```

---

# 13. Why is JWT called stateless?

In traditional session authentication:

```text
Client
  ↓
Session ID
  ↓
Server stores session
```

With JWT:

```text
Client
  ↓
JWT containing signed claims
  ↓
Server validates token
```

The server doesn't need to maintain a traditional login session for every client.

Therefore JWT-based authentication is commonly used for **stateless APIs**.

### Important nuance

JWT itself doesn't magically make an application stateless.

If you store server-side session state associated with the token, you've introduced state again.

---

# 14. Where should JWT be sent?

Typically:

```http
Authorization: Bearer <token>
```

Example:

```http
Authorization: Bearer eyJhbGciOi...
```

Spring Security's authentication mechanism/filter can extract and validate it.

---

# 15. What is `OncePerRequestFilter`?

Very commonly used when implementing a JWT filter.

```java
public class JwtAuthenticationFilter
        extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(...) {

        // extract JWT
        // validate JWT
        // set SecurityContext

        filterChain.doFilter(request, response);
    }
}
```

It is useful when you want your custom filter to execute once per request dispatch.

---

# 16. Where do we put the JWT filter?

Typically before the appropriate authentication filter:

```java
http.addFilterBefore(
    jwtAuthenticationFilter,
    UsernamePasswordAuthenticationFilter.class
);
```

Why?

Because you want to process the JWT and establish authentication before authorization decisions are made.

---

# 17. What are Roles and Authorities?

Both represent permissions used by Spring Security, but conceptually:

### Authority

Specific permission:

```text
ACCOUNT_READ
ACCOUNT_WRITE
ACCOUNT_DELETE
```

### Role

Higher-level grouping:

```text
ROLE_ADMIN
ROLE_USER
```

For example:

```text
ADMIN
  ↓
ACCOUNT_READ
ACCOUNT_WRITE
ACCOUNT_DELETE
```

---

# 18. `hasRole()` vs `hasAuthority()`

Very common trap.

```java
.hasRole("ADMIN")
```

Spring conventionally checks for:

```text
ROLE_ADMIN
```

Whereas:

```java
.hasAuthority("ADMIN")
```

checks exactly:

```text
ADMIN
```

So:

```java
hasRole("ADMIN")
```

≈

```java
hasAuthority("ROLE_ADMIN")
```

assuming the standard role prefix is being used.

### Interview trap

If you have:

```text
ROLE_ADMIN
```

then:

```java
hasRole("ADMIN")
```

not:

```java
hasRole("ROLE_ADMIN")
```

---

# 19. What is `permitAll()`?

Example:

```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers("/login").permitAll()
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated()
)
```

Means:

```text
/login       → public
/admin/**    → ADMIN only
everything   → authenticated
```

---

# 20. What is `authenticated()`?

```java
.anyRequest().authenticated()
```

means the request must have a successfully authenticated principal.

It does **not** mean the user must be ADMIN.

For example:

```text
USER  → allowed
ADMIN → allowed
```

assuming both are authenticated.

---

# 21. What is `@PreAuthorize`?

Allows method-level authorization.

Example:

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteAccount(Long id) {
    ...
}
```

Or:

```java
@PreAuthorize("hasAuthority('ACCOUNT_WRITE')")
public void updateAccount(...) {
}
```

Usually enabled with:

```java
@EnableMethodSecurity
```

This is useful because authorization can be placed close to the business operation rather than only at URL level.

---

# 22. URL-level vs Method-level security

### URL-level

```java
.requestMatchers("/admin/**")
.hasRole("ADMIN")
```

Protects HTTP endpoints.

### Method-level

```java
@PreAuthorize("hasRole('ADMIN')")
```

Protects the method itself.

You can use both.

---

# 23. What is CSRF?

CSRF = **Cross-Site Request Forgery**.

An attacker attempts to make a user's browser send an unwanted authenticated request.

It is especially relevant when authentication relies on **automatically sent browser credentials such as cookies**.

For a stateless API using an `Authorization: Bearer` header, CSRF considerations are different because browsers don't automatically attach that header to cross-site requests.

### Interview answer

> CSRF protects against forged requests made using a victim's automatically supplied authentication credentials. Whether it should be enabled depends on the authentication mechanism and application architecture.

---

# 24. CORS vs CSRF

This is frequently asked.

### CORS

Controls:

> Which origins are allowed to make browser requests to my API?

Example:

```text
frontend.com
     ↓
api.com
```

### CSRF

Protects against:

> A malicious site causing a user's browser to perform an unwanted authenticated action.

Simple memory:

```text
CORS → Cross-origin browser access

CSRF → Forged authenticated request
```

---

# 🔥 Spring Security request flow to memorize

If interviewer asks:

> "Explain JWT authentication in Spring Security."

Say:

```text
Client
  ↓
Authorization: Bearer JWT
  ↓
Security Filter Chain
  ↓
JWT Filter
  ↓
Extract token
  ↓
Validate signature + expiry
  ↓
Extract username/claims
  ↓
Create Authentication
  ↓
SecurityContextHolder
  ↓
Authorization
  ↓
Controller
```

That answer alone covers a **large chunk of Spring Security interview questions**.

---

## Next Spring Security section

We'll quickly cover:

* Access token vs refresh token
* JWT expiration
* Token validation
* OAuth2
* Resource Server
* Authentication vs authorization flow
* 401 vs 403
* Session vs JWT
* BCrypt
* Security filter ordering
* CORS configuration
* common Spring Security traps

Then we'll move straight to **Spring AOP**.
