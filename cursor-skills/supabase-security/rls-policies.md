# RLS Policy Patterns

## Basic Patterns

### User-Owned Data

```sql
-- Table with user_id column
CREATE POLICY "Users manage own data" ON items
  FOR ALL USING (auth.uid() = user_id);
```

### Organization/Team Access

```sql
-- Users access data from their organization
CREATE POLICY "Org members access" ON documents
  FOR SELECT USING (
    org_id IN (
      SELECT org_id FROM org_members 
      WHERE user_id = auth.uid()
    )
  );
```

### Role-Based Access

```sql
-- Check user role from metadata or profiles table
CREATE POLICY "Admins full access" ON settings
  FOR ALL USING (
    EXISTS (
      SELECT 1 FROM profiles
      WHERE id = auth.uid() AND role = 'admin'
    )
  );

CREATE POLICY "Users read only" ON settings
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM profiles
      WHERE id = auth.uid() AND role = 'user'
    )
  );
```

---

## Advanced Patterns

### Shared Resources

```sql
-- Owner or explicitly shared
CREATE POLICY "Owner or shared" ON files
  FOR SELECT USING (
    owner_id = auth.uid() OR
    id IN (
      SELECT file_id FROM file_shares
      WHERE shared_with = auth.uid()
    )
  );
```

### Hierarchical Access

```sql
-- Access based on folder hierarchy
CREATE POLICY "Folder access" ON files
  FOR SELECT USING (
    folder_id IN (
      SELECT id FROM folders
      WHERE owner_id = auth.uid()
      UNION
      SELECT folder_id FROM folder_shares
      WHERE user_id = auth.uid()
    )
  );
```

### Time-Based Access

```sql
-- Only access during valid period
CREATE POLICY "Valid subscription" ON premium_content
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM subscriptions
      WHERE user_id = auth.uid()
      AND starts_at <= now()
      AND ends_at > now()
    )
  );
```

### Public + Private

```sql
-- Public items visible to all, private only to owner
CREATE POLICY "Public or own" ON posts
  FOR SELECT USING (
    is_public = true OR author_id = auth.uid()
  );
```

---

## Insert/Update Patterns

### Validate on Insert

```sql
-- Ensure user_id is set to current user
CREATE POLICY "Insert own" ON items
  FOR INSERT WITH CHECK (
    user_id = auth.uid()
  );
```

### Prevent Field Modification

```sql
-- Can update, but can't change owner
CREATE POLICY "Update own" ON items
  FOR UPDATE USING (owner_id = auth.uid())
  WITH CHECK (owner_id = auth.uid());
```

### Audit Trail

```sql
-- Ensure created_by is set correctly
CREATE POLICY "Insert with audit" ON records
  FOR INSERT WITH CHECK (
    created_by = auth.uid() AND
    created_at = now()
  );
```

---

## Common Vulnerabilities

### Missing Policy (Allows Nothing)

```sql
-- RLS enabled but no policy = blocks all access
ALTER TABLE items ENABLE ROW LEVEL SECURITY;
-- Need at least one policy!
```

### Overly Permissive

```sql
-- 🔴 DANGEROUS - Allows everything
CREATE POLICY "bad" ON items FOR ALL USING (true);

-- ✅ FIXED - Restrict to owner
CREATE POLICY "good" ON items FOR ALL USING (auth.uid() = user_id);
```

### Service Role Bypass

```sql
-- Service role bypasses RLS by default!
-- In app code, never use service role on client
-- Use: supabase.rpc('function', {}, { headers: { ... } })
```

### Missing Auth Check

```sql
-- 🔴 DANGEROUS - No auth verification
CREATE POLICY "bad" ON items FOR SELECT USING (status = 'public');

-- ✅ FIXED - Still require authentication
CREATE POLICY "good" ON items FOR SELECT USING (
  auth.uid() IS NOT NULL AND status = 'public'
);
```

---

## Testing RLS

### Test as Specific User

```sql
-- In Supabase SQL editor, test as user
SET request.jwt.claims = '{"sub": "user-uuid-here"}';
SELECT * FROM your_table; -- See what user would see
```

### Test Policy Logic

```sql
-- Create test function
CREATE OR REPLACE FUNCTION test_rls(test_user_id uuid)
RETURNS SETOF your_table AS $$
BEGIN
  PERFORM set_config('request.jwt.claims', 
    json_build_object('sub', test_user_id)::text, true);
  RETURN QUERY SELECT * FROM your_table;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

---

## Performance Considerations

### Index for RLS

```sql
-- Add index for commonly filtered columns in RLS
CREATE INDEX idx_items_user_id ON items(user_id);
CREATE INDEX idx_org_members_user_id ON org_members(user_id);
```

### Avoid N+1 in Policies

```sql
-- 🔴 SLOW - Subquery per row
CREATE POLICY "slow" ON items FOR SELECT USING (
  (SELECT role FROM profiles WHERE id = auth.uid()) = 'admin'
);

-- ✅ FAST - Use EXISTS
CREATE POLICY "fast" ON items FOR SELECT USING (
  EXISTS (SELECT 1 FROM profiles WHERE id = auth.uid() AND role = 'admin')
);
```
