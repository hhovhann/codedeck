# OWASP Top 10

The ten most critical web application security risks, based on the [OWASP Top 10 (2021)](https://owasp.org/Top10/). Every developer should know these by heart.

> **License:** The OWASP Top 10 is published by the [OWASP Foundation](https://owasp.org) under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). This deck contains original educational summaries and original code examples inspired by the OWASP Top 10 categories. The content in this file is shared under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

---

## 1. Broken Access Control
**Advice:** Enforce that users can only act within their intended permissions. Every endpoint must verify the user is authorized for that specific resource, not just authenticated.
**Bad:**
```java
@GetMapping("/api/users/{id}/orders")
public List<Order> getOrders(@PathVariable Long id) {
    return orderRepository.findByUserId(id); // anyone can view any user's orders
}
```
**Good:**
```java
@GetMapping("/api/users/{id}/orders")
public List<Order> getOrders(@PathVariable Long id, Authentication auth) {
    if (!auth.getName().equals(userService.findById(id).getUsername())) {
        throw new AccessDeniedException("Not your resource");
    }
    return orderRepository.findByUserId(id);
}
```
**Commit check:** Can a user access or modify another user's data through this endpoint?

## 2. Cryptographic Failures
**Advice:** Protect sensitive data in transit and at rest. Never store passwords in plain text, use weak hashing, or transmit secrets over unencrypted channels.
**Bad:**
```java
user.setPassword(rawPassword); // stored in plain text
String encoded = Base64.getEncoder().encodeToString(secret.getBytes()); // encoding != encryption
```
**Good:**
```java
user.setPassword(passwordEncoder.encode(rawPassword)); // bcrypt/argon2
// Use TLS for transit, AES-256 for storage, never roll your own crypto
```
**Commit check:** Am I storing or transmitting sensitive data without proper encryption or hashing?

## 3. Injection
**Advice:** Never trust user input. Use parameterized queries, prepared statements, or ORM frameworks. This applies to SQL, LDAP, OS commands, ORM queries, and expression languages.
**Bad:**
```java
String query = "SELECT * FROM users WHERE name = '" + userName + "'";
Runtime.getRuntime().exec("ping " + userInput);
```
**Good:**
```java
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE name = ?");
ps.setString(1, userName);
// For MyBatis: use #{param} not ${param}
```
**Commit check:** Does any code concatenate user input into SQL, commands, or queries?

## 4. Insecure Design
**Advice:** Security must be built into the design, not bolted on after. Threat model your features. Consider: what happens if the user sends 10,000 requests? What if they skip step 2 of 3?
**Bad:** A password reset that asks security questions answerable from social media, with no rate limiting.
**Good:** Multi-factor verification, rate limiting, secure token-based reset with expiry, account lockout after failed attempts.
**Commit check:** Did I consider abuse scenarios for this feature? What's the worst a malicious user could do?

## 5. Security Misconfiguration
**Advice:** Default configurations are often insecure. Disable unnecessary features, remove default accounts, configure error handling to not leak stack traces, keep frameworks patched.
**Bad:**
```yaml
spring:
  h2:
    console:
      enabled: true  # H2 console exposed in production
server:
  error:
    include-stacktrace: always  # stack traces leak to users
```
**Good:**
```yaml
spring:
  h2:
    console:
      enabled: false
server:
  error:
    include-stacktrace: never
    include-message: never
```
**Commit check:** Am I deploying with debug features, default credentials, or verbose error messages enabled?

## 6. Vulnerable and Outdated Components
**Advice:** Every library you use is an attack surface. Track dependencies, monitor for CVEs, remove unused libraries, and update regularly.
**Bad:** Using Log4j 2.14.1 (CVE-2021-44228), or any library with known critical vulnerabilities.
**Good:** Run `./gradlew dependencyCheckAnalyze` or `npm audit` regularly. Pin versions. Subscribe to security advisories for your key dependencies.
**Commit check:** Are my dependencies current? Did I check for known vulnerabilities?

## 7. Identification and Authentication Failures
**Advice:** Implement robust authentication. Use strong password policies, multi-factor authentication, and session management. Protect against brute force and credential stuffing.
**Bad:**
```java
if (password.length() >= 4) { /* accept */ } // weak policy
session.setMaxInactiveInterval(86400 * 30); // 30-day sessions
```
**Good:**
```java
// Minimum 8 chars, complexity requirements, breached password check
// Short session timeout, re-auth for sensitive operations
// Rate limiting on login, account lockout, MFA
```
**Commit check:** Does my authentication allow weak passwords, long sessions, or unlimited login attempts?

## 8. Software and Data Integrity Failures
**Advice:** Verify the integrity of software updates, critical data, and CI/CD pipelines. Use digital signatures, checksums, and trusted sources.
**Bad:**
```java
// Deserializing untrusted data without validation
Object obj = new ObjectInputStream(untrustedStream).readObject();
// Pulling dependencies without checksum verification
```
**Good:**
```java
// Use allowlists for deserialization
// Verify dependency checksums (Gradle verification-metadata.xml)
// Sign releases, verify CI/CD pipeline integrity
```
**Commit check:** Am I deserializing untrusted data? Are my build dependencies verified?

## 9. Security Logging and Monitoring Failures
**Advice:** Log all authentication events, access control failures, and input validation failures. Ensure logs are monitored and alerts are configured for suspicious activity.
**Bad:**
```java
catch (AuthenticationException e) {
    return ResponseEntity.status(401).build(); // no logging
}
```
**Good:**
```java
catch (AuthenticationException e) {
    log.warn("Failed login attempt for user={} from ip={}", username, request.getRemoteAddr());
    securityAuditService.recordFailedLogin(username, request);
    return ResponseEntity.status(401).build();
}
```
**Commit check:** Am I logging security-relevant events? Would I notice if someone was attacking this?

## 10. Server-Side Request Forgery (SSRF)
**Advice:** Never fetch remote resources based on user-supplied URLs without validation. Attackers can use this to access internal services, cloud metadata, or local files.
**Bad:**
```java
@GetMapping("/fetch")
public String fetch(@RequestParam String url) {
    return restTemplate.getForObject(url, String.class); // user controls URL
}
```
**Good:**
```java
@GetMapping("/fetch")
public String fetch(@RequestParam String url) {
    URI uri = URI.create(url);
    if (!ALLOWED_HOSTS.contains(uri.getHost())) {
        throw new SecurityException("Host not allowed: " + uri.getHost());
    }
    return restTemplate.getForObject(uri, String.class);
}
```
**Commit check:** Does any code make HTTP requests to user-supplied URLs without validating the destination?