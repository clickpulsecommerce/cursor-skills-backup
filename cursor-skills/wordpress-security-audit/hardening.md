# WordPress Security Hardening

Post-remediation security hardening recommendations.

## File Permissions

**Correct Permissions:**
- Files: `644` (rw-r--r--)
- Directories: `755` (rwxr-xr-x)
- wp-config.php: `600` (rw-------)

```bash
find . -type f -exec chmod 644 {} \;
find . -type d -exec chmod 755 {} \;
chmod 600 wp-config.php
```

## .htaccess Security

**Protect wp-config.php:**
```apache
<Files wp-config.php>
    Order allow,deny
    Deny from all
</Files>
```

**Disable PHP execution in uploads:**
```apache
<Directory wp-content/uploads>
    php_flag engine off
</Directory>
```

**Prevent directory browsing:**
```apache
Options -Indexes
```

## wp-config.php Hardening

**Security Keys:**
- Ensure unique, strong security keys
- Regenerate if compromised

**Disable File Editing:**
```php
define('DISALLOW_FILE_EDIT', true);
```

**Limit Database Access:**
- Use database user with minimal privileges
- Separate user for WordPress vs. other applications

**Disable PHP Error Display:**
```php
define('WP_DEBUG', false);
define('WP_DEBUG_DISPLAY', false);
@ini_set('display_errors', 0);
```

## Core File Verification

**Verify WordPress Core:**
```bash
# Download checksums
wget https://api.wordpress.org/core/checksums/1.0/?version=X.X.X

# Compare files
wp core verify-checksums
```

## Plugin & Theme Security

**Best Practices:**
- Keep plugins/themes updated
- Remove unused plugins/themes
- Only install from trusted sources
- Review plugin code before installation

## Database Security

**Check for Malicious Entries:**
```sql
-- Check active plugins
SELECT * FROM wp_options WHERE option_name = 'active_plugins';

-- Check cron jobs
SELECT * FROM wp_options WHERE option_name = 'cron';

-- Check for suspicious posts
SELECT * FROM wp_posts WHERE post_content LIKE '%eval%' OR post_content LIKE '%base64_decode%';
```

## Monitoring

**File Integrity Monitoring:**
- Monitor core WordPress files for changes
- Alert on new PHP files in uploads directory
- Track modifications to wp-config.php

**Log Analysis:**
- Review access logs for suspicious patterns
- Monitor failed login attempts
- Check for unusual file access patterns

## Regular Maintenance

**Weekly:**
- Review security logs
- Check for plugin/theme updates
- Verify file permissions

**Monthly:**
- Full security scan
- Database audit
- Backup verification

**After Security Incident:**
- Change all passwords
- Regenerate security keys
- Review and update all plugins/themes
- Implement additional security measures