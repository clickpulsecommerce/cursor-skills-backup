---
name: supabase-security
description: Perform security audits for Next.js apps with Supabase. Analyze code for vulnerabilities, review and fix RLS policies, check authentication flows, validate API endpoints, and prevent SQL injection and XSS attacks. Use when the user asks for security review, security audit, RLS check, vulnerability scan, or mentions security concerns.
---

# Supabase Security Audit

## Quick Start

When performing a security audit:

1. **Copy the checklist** below and track progress
2. **Run each check** systematically
3. **Document findings** with severity levels
4. **Fix critical issues** immediately

## Audit Checklist

```
Security Audit Progress:
- [ ] 1. RLS Policies Review
- [ ] 2. Authentication Security
- [ ] 3. API Endpoint Security
- [ ] 4. Environment Variables
- [ ] 5. Client-Side Security
- [ ] 6. Storage Security
- [ ] 7. Edge Functions Security
- [ ] 8. Realtime Security
```

---

## 1. RLS Policies Review

### Check for Missing RLS

```sql
-- Find tables without RLS enabled
SELECT schemaname, tablename 
FROM pg_tables 
WHERE schemaname = 'public' 
AND tablename NOT IN (
  SELECT tablename FROM pg_tables t
  JOIN pg_class c ON c.relname = t.tablename
  WHERE c.relrowsecurity = true
);
```

### Common RLS Vulnerabilities

| Vulnerability | Check | Fix |
|--------------|-------|-----|
| Missing policies | Table has RLS but no policies | Add appropriate SELECT/INSERT/UPDATE/DELETE policies |
| Overly permissive | `USING (true)` without conditions | Add user-specific conditions |
| Missing auth check | No `auth.uid()` verification | Add `auth.uid() = user_id` |
| Bypass via service role | Client using service key | Use anon key on client |

### Secure RLS Pattern

```sql
-- Enable RLS
ALTER TABLE your_table ENABLE ROW LEVEL SECURITY;

-- User can only see own data
CREATE POLICY "Users read own data" ON your_table
  FOR SELECT USING (auth.uid() = user_id);

-- User can only insert own data
CREATE POLICY "Users insert own data" ON your_table
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- User can only update own data
CREATE POLICY "Users update own data" ON your_table
  FOR UPDATE USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- User can only delete own data
CREATE POLICY "Users delete own data" ON your_table
  FOR DELETE USING (auth.uid() = user_id);
```

For detailed RLS patterns, see [rls-policies.md](rls-policies.md).

---

## 2. Authentication Security

### Check Auth Configuration

Search codebase for these issues:

```
🔴 CRITICAL:
- Hardcoded SUPABASE_SERVICE_ROLE_KEY in client code
- Service role key in NEXT_PUBLIC_ variables
- Missing auth checks in API routes

🟡 WARNING:
- No session refresh handling
- Missing error handling on auth operations
- Exposed user metadata without filtering
```

### Secure Auth Pattern (Next.js)

```typescript
// middleware.ts - Protect routes
import { createServerClient } from '@supabase/ssr'
import { NextResponse } from 'next/server'

export async function middleware(request) {
  const supabase = createServerClient(/* config */)
  const { data: { session } } = await supabase.auth.getSession()
  
  if (!session && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  
  return NextResponse.next()
}
```

---

## 3. API Endpoint Security

### Audit All API Routes

For each `/app/api/**/route.ts` or `/pages/api/**`:

```
□ Validates authentication
□ Validates authorization (user can access resource)
□ Validates input (type, length, format)
□ Sanitizes output
□ Has rate limiting consideration
□ Returns appropriate error codes (not 500 with details)
```

### Secure API Pattern

```typescript
// app/api/resource/route.ts
import { createRouteHandlerClient } from '@supabase/auth-helpers-nextjs'

export async function POST(request: Request) {
  const supabase = createRouteHandlerClient({ cookies })
  
  // 1. Verify authentication
  const { data: { user }, error: authError } = await supabase.auth.getUser()
  if (authError || !user) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }
  
  // 2. Validate input
  const body = await request.json()
  const validated = schema.safeParse(body) // Use zod
  if (!validated.success) {
    return Response.json({ error: 'Invalid input' }, { status: 400 })
  }
  
  // 3. Check authorization (RLS handles this, but double-check for sensitive ops)
  // 4. Perform operation
  // 5. Return sanitized response
}
```

---

## 4. Environment Variables

### Check for Exposed Secrets

```bash
# Search for exposed secrets
grep -r "SUPABASE_SERVICE_ROLE" --include="*.ts" --include="*.tsx" --include="*.js"
grep -r "eyJ" --include="*.ts" --include="*.tsx" --include="*.js"  # JWT tokens
```

### Required Variables Structure

```
✅ PUBLIC (can be in NEXT_PUBLIC_):
- NEXT_PUBLIC_SUPABASE_URL
- NEXT_PUBLIC_SUPABASE_ANON_KEY

🔒 SERVER ONLY (never NEXT_PUBLIC_):
- SUPABASE_SERVICE_ROLE_KEY
- DATABASE_URL
- Any API secrets
```

---

## 5. Client-Side Security

### XSS Prevention

Search for dangerous patterns:

```typescript
// 🔴 DANGEROUS - Never do this
dangerouslySetInnerHTML={{ __html: userInput }}
element.innerHTML = userInput

// ✅ SAFE - Use text content or sanitize
<div>{userContent}</div>
import DOMPurify from 'dompurify'
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }} />
```

### SQL Injection Prevention

```typescript
// 🔴 DANGEROUS - String concatenation
const { data } = await supabase
  .from('users')
  .select('*')
  .filter('name', 'eq', `'${userInput}'`) // VULNERABLE

// ✅ SAFE - Parameterized (Supabase client handles this)
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('name', userInput) // Safe - parameterized
```

---

## 6. Storage Security

### Check Storage Policies

```sql
-- Review storage policies
SELECT * FROM storage.policies;
```

### Secure Storage Pattern

```sql
-- Users can only access own files
CREATE POLICY "Users access own files" ON storage.objects
  FOR ALL USING (
    bucket_id = 'user-files' AND
    (storage.foldername(name))[1] = auth.uid()::text
  );
```

### File Upload Validation

```typescript
// Validate file type and size
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'application/pdf']
const MAX_SIZE = 5 * 1024 * 1024 // 5MB

if (!ALLOWED_TYPES.includes(file.type)) {
  throw new Error('Invalid file type')
}
if (file.size > MAX_SIZE) {
  throw new Error('File too large')
}
```

---

## 7. Edge Functions Security

### Check Edge Function Auth

```typescript
// supabase/functions/my-function/index.ts
import { createClient } from '@supabase/supabase-js'

Deno.serve(async (req) => {
  // 1. Verify JWT
  const authHeader = req.headers.get('Authorization')
  if (!authHeader) {
    return new Response('Unauthorized', { status: 401 })
  }
  
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_ANON_KEY')!,
    { global: { headers: { Authorization: authHeader } } }
  )
  
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) {
    return new Response('Unauthorized', { status: 401 })
  }
  
  // Continue with authorized request...
})
```

---

## 8. Realtime Security

### Check Realtime Policies

Realtime respects RLS policies. Ensure:

```
□ RLS is enabled on tables with realtime
□ Broadcast policies are restrictive
□ Presence channel access is controlled
```

---

## Severity Classification

| Level | Icon | Criteria | Action |
|-------|------|----------|--------|
| Critical | 🔴 | Data breach possible, auth bypass | Fix immediately |
| High | 🟠 | Privilege escalation, data leak | Fix before deploy |
| Medium | 🟡 | Security best practice violation | Plan fix |
| Low | 🟢 | Minor improvement | Optional |

---

## Report Template

After audit, provide findings in this format:

```markdown
# Security Audit Report

## Summary
- Critical: X issues
- High: X issues  
- Medium: X issues
- Low: X issues

## Critical Findings

### [Finding Title]
**Location:** `path/to/file.ts:line`
**Issue:** Description of the vulnerability
**Risk:** What could happen if exploited
**Fix:** Specific code/config change needed
```

---

## Additional Resources

- [RLS Policy Patterns](rls-policies.md) - Comprehensive RLS examples
- [Security Checklist](security-checklist.md) - Full audit checklist
