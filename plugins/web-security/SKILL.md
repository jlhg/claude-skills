---
name: Web Application Security
description: Web application security best practices covering API rate limiting, path traversal protection, and authentication architecture design. Use when implementing DDoS protection, securing sensitive paths, designing token management systems, or when user mentions Rack::Attack, Cloudflare WAF, authentication, rate limiting, token, Redis, PostgreSQL, security, API protection, or path traversal.
---

# Web Application Security

## Overview

Comprehensive web application security guidelines covering API protection, authentication architecture, and malicious attack prevention. These best practices are primarily for Rails API applications but are applicable to other web frameworks.

## Core Security Practices

### Rate Limiting

Critical defense mechanism against API abuse and DDoS attacks. See [Rate Limiting Guide](references/rate-limiting.md) for:

- **Rack::Attack Configuration**
  - Throttling strategies by IP, User, Endpoint
  - Allowlist and blocklist management
  - Progressive throttling
  - Custom response messages

- **Cloudflare WAF**
  - Global rate limiting rules
  - Geographic blocking
  - Bot protection
  - DDoS protection levels

- **Redis Storage Strategy**
  - Using redis_cache instance
  - Memory management
  - Performance considerations

- **Monitoring and Alerting**
  - Tracking throttle events
  - Analyzing attack patterns
  - Adjusting throttle thresholds

### Path Security

Protection against path traversal and malicious path scanning attacks. See [Security Rules Guide](references/security-rules.md) for:

- **Common Attack Patterns**
  - `.env` file probing
  - `.git` directory access
  - Backup file scanning
  - Admin interface detection

- **Rack::Attack Protection Rules**
  - Blocking malicious path patterns
  - Custom 403 responses
  - Logging and tracking

- **Cloudflare WAF Rules**
  - Global path blocking
  - Custom security rules
  - Alert configuration

- **Best Practices**
  - Never put sensitive files in public/
  - Use .gitignore to prevent accidental commits
  - Regularly review WAF logs

### Authentication Architecture

Secure and scalable authentication system design. See [Authentication Guide](references/authentication.md) for:

- **Token Management Strategy**
  - Hybrid Redis (cache) + PostgreSQL (source of truth) architecture
  - Why not pure JWT
  - Why not pure Redis
  - Expand-Contract pattern

- **Token Storage**
  - Never store plaintext tokens
  - Use SHA256 digest
  - Redis key design
  - PostgreSQL schema design

- **Token Verification Flow**
  - Fast path: Redis cache lookup
  - Slow path: PostgreSQL fallback
  - Automatic cache warming
  - Cache hit rate optimization

- **Token Revocation**
  - Immediate revocation (Redis deletion)
  - Audit trail (PostgreSQL marking)
  - Batch revocation (password change, suspicious activity)

- **Frontend Token Storage**
  - Browser storage options comparison (localStorage vs sessionStorage vs cookies vs memory)
  - Recommended: httpOnly cookie implementation
  - High-security: memory-only storage with refresh tokens
  - OAuth 2.0 integration patterns
  - CORS configuration for cookie-based auth
  - Frontend security checklist
  - Handling concurrent requests (race condition prevention, request queue management)

- **Security Best Practices**
  - Token entropy (256-bit)
  - Short expiration time
  - Refresh token pattern
  - IP and User-Agent tracking
  - Rate limiting integration

## Security Checklist

Checklist for implementing security protections:

### Rate Limiting
- [ ] Configure appropriate global rate limits
- [ ] Strengthen throttling for sensitive operations (login, registration)
- [ ] Configure allowlist (e.g., internal IPs)
- [ ] Set up monitoring and alerts
- [ ] Regularly review Rack::Attack logs

### Path Security
- [ ] Block common malicious path patterns
- [ ] Ensure sensitive files are not in public/
- [ ] Verify .gitignore completeness
- [ ] Regularly review WAF logs
- [ ] Test that malicious path requests are properly blocked

### Authentication
- [ ] Store tokens using SHA256 digest
- [ ] Use Redis + PostgreSQL hybrid architecture
- [ ] Set appropriate token expiration times
- [ ] Implement refresh token mechanism
- [ ] Revoke all tokens on password change
- [ ] Track IP and User-Agent
- [ ] Integrate rate limiting
- [ ] Use HTTPS only for sensitive information

## Common Vulnerabilities

### DDoS and API Abuse
- **Problem**: Unthrottled APIs are vulnerable to abuse
- **Solution**: Multi-layer rate limiting (Cloudflare + Rack::Attack)
- **Reference**: [Rate Limiting Guide](references/rate-limiting.md)

### Path Traversal
- **Problem**: Attackers attempt to access sensitive files
- **Solution**: WAF rules + Rack::Attack blocking
- **Reference**: [Security Rules Guide](references/security-rules.md)

### Token Theft
- **Problem**: Stolen tokens remain valid
- **Solution**: Immediate revocation mechanism + short-lived tokens
- **Reference**: [Authentication Guide](references/authentication.md)

### Brute Force Attacks
- **Problem**: Brute force login attempts
- **Solution**: Login rate limiting + IP blocking
- **Reference**: [Rate Limiting Guide](references/rate-limiting.md)

## Resources

- [Rate Limiting Guide](references/rate-limiting.md) - API rate limiting with Rack::Attack and Cloudflare WAF
- [Security Rules Guide](references/security-rules.md) - Path traversal protection and malicious path blocking
- [Authentication Guide](references/authentication.md) - Secure authentication architecture with Redis and PostgreSQL

## Related Skills

- **ruby-development**: RuboCop rules for secure code patterns
- **performance-optimization**: Rate limiting performance considerations
- **backend-architecture**: Redis and PostgreSQL architecture for authentication
