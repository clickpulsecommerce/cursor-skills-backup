# Complete Security Checklist

## Database Security

### RLS Policies
- [ ] All tables have RLS enabled
- [ ] Every table has appropriate SELECT policy
- [ ] Every table has appropriate INSERT policy with WITH CHECK
- [ ] Every table has appropriate UPDATE policy with USING and WITH CHECK
- [ ] Every table has appropriate DELETE policy
- [ ] No `USING (true)` policies without additional restrictions
- [ ] All policies verify `auth.uid()`
- [ ] Service role not exposed to client

### Data Validation
- [ ] Database constraints (NOT NULL, CHECK, UNIQUE)
- [ ] Foreign key constraints enforced
- [ ] Enum types for restricted values
- [ ] Trigger validation for complex rules

---

## Authentication

### Configuration
- [ ] Strong password policy configured
- [ ] Email confirmation required (if applicable)
- [ ] Rate limiting on auth endpoints
- [ ] Secure session configuration

### Implementation
- [ ] All protected routes check authentication
- [ ] Session refresh handled properly
- [ ] Logout clears all session data
- [ ] Auth errors don't leak information

### Token Security
- [ ] JWT expiry is reasonable (default: 1 hour)
- [ ] Refresh tokens handled securely
- [ ] No tokens stored in localStorage (use cookies)
- [ ] HttpOnly cookies where possible

---

## API Security

### Route Protection
- [ ] All API routes verify authentication
- [ ] Authorization checked (user can access resource)
- [ ] Input validated with schema (zod/yup)
- [ ] Output sanitized

### Error Handling
- [ ] Generic error messages to client
- [ ] Detailed errors only in logs
- [ ] No stack traces in production
- [ ] Appropriate HTTP status codes

### Rate Limiting
- [ ] Rate limiting on sensitive endpoints
- [ ] Brute force protection on login
- [ ] API abuse prevention

---

## Client Security

### XSS Prevention
- [ ] No `dangerouslySetInnerHTML` with user input
- [ ] No `innerHTML` with user input
- [ ] User content sanitized if HTML needed
- [ ] Content-Security-Policy headers set

### Data Exposure
- [ ] No sensitive data in client bundle
- [ ] No service keys in client code
- [ ] User data filtered before display
- [ ] No excessive data fetching

---

## Environment & Secrets

### Environment Variables
- [ ] Service role key not in NEXT_PUBLIC_
- [ ] All secrets in .env (not committed)
- [ ] .env in .gitignore
- [ ] Production secrets in secure vault

### Key Management
- [ ] Anon key used for client
- [ ] Service key only on server
- [ ] API keys rotated periodically
- [ ] Old keys revoked

---

## Storage Security

### Bucket Policies
- [ ] Storage RLS policies configured
- [ ] Public buckets intentional and documented
- [ ] File paths include user ID
- [ ] No directory traversal possible

### Upload Validation
- [ ] File type validation
- [ ] File size limits
- [ ] Filename sanitization
- [ ] Virus scanning (if applicable)

---

## Edge Functions

### Authentication
- [ ] JWT verified on all functions
- [ ] User context established correctly
- [ ] No bypassing of auth checks

### Security
- [ ] CORS configured correctly
- [ ] Input validated
- [ ] Secrets from environment only
- [ ] Error responses sanitized

---

## Realtime

### Channel Security
- [ ] RLS enforced on realtime tables
- [ ] Broadcast policies restrictive
- [ ] Presence requires authentication
- [ ] No sensitive data in realtime payloads

---

## Infrastructure

### HTTPS
- [ ] All connections over HTTPS
- [ ] HSTS enabled
- [ ] Secure cookies only

### Headers
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options: DENY
- [ ] Content-Security-Policy configured
- [ ] Referrer-Policy set

### Logging
- [ ] Security events logged
- [ ] No sensitive data in logs
- [ ] Logs monitored for anomalies
- [ ] Audit trail for sensitive operations

---

## Code Quality

### Dependencies
- [ ] Dependencies up to date
- [ ] No known vulnerabilities (npm audit)
- [ ] Lockfile committed
- [ ] Minimal dependencies

### Code Review
- [ ] Security-focused code review
- [ ] No hardcoded credentials
- [ ] No commented-out security code
- [ ] Consistent security patterns

---

## Quick Commands

```bash
# Check for exposed secrets
grep -rn "SUPABASE_SERVICE_ROLE\|sk_live\|eyJ" --include="*.ts" --include="*.tsx" .

# Audit dependencies
npm audit
pnpm audit

# Find dangerouslySetInnerHTML
grep -rn "dangerouslySetInnerHTML" --include="*.tsx" --include="*.jsx" .

# Find potential SQL injection
grep -rn "\.rpc\|\.from\|raw\(" --include="*.ts" . | grep -v node_modules
```
