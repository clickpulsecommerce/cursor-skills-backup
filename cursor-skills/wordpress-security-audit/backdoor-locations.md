# Common WordPress Backdoor Locations

Typical locations where malware and backdoors are found in WordPress installations.

## Root Directory

**Suspicious Files:**
- Random alphanumeric names: `aB3xK9m.php`, `Yi42JCOhH71.php`
- Files with timestamps: `index.php__10ac534`
- Hidden files: `.backdoor.php`, `.shell.php`
- Legitimate-looking names: `wp-config-backup.php`, `wp-settings-backup.php`

## wp-content Directory

### wp-content/uploads/
- PHP files (should only contain media files)
- Suspicious directories with random names
- `.php` files disguised as images

### wp-content/themes/
- Modified theme files with injected code
- Unknown theme directories
- `functions.php` with malicious code
- `header.php` or `footer.php` with hidden scripts

### wp-content/plugins/
- Unknown or suspicious plugins
- Modified legitimate plugin files
- Plugins with random names

### wp-content/cache/
- PHP files in cache directories
- Suspicious cached content

## Core WordPress Files

**Commonly Modified:**
- `wp-config.php` - Database credentials, malicious includes
- `wp-load.php` - Entry point for backdoors
- `wp-settings.php` - Core settings manipulation
- `index.php` - Root index file injection
- `wp-includes/functions.php` - Core function modifications

## Hidden Directories

**Common Locations:**
- `/.well-known/` - Sometimes used for hidden files
- `/tmp/` - Temporary backdoor storage
- `/cache/` - Hidden cache directories
- Directories starting with `.` - Hidden from normal listing

## Database Backdoors

**wp_options Table:**
- Malicious code in `active_plugins` option
- Injected code in `cron` option
- Modified `siteurl` or `home` options

**wp_posts Table:**
- Hidden posts with malicious code
- Injected scripts in post content

## .htaccess Modifications

**Common Injections:**
- Unauthorized redirects
- File access restrictions hiding backdoors
- Rewrite rules pointing to malicious files

## File Permissions

**Red Flags:**
- Files with 0777 permissions
- Directories with 0777 permissions
- Recently modified files with suspicious permissions

## Detection Commands

```bash
# Find PHP files in uploads (shouldn't exist)
find wp-content/uploads -name "*.php"

# Find files with suspicious names
find . -name "*.php" -type f | grep -E "^[a-zA-Z0-9]{8,}\.php$"

# Find recently modified files
find . -name "*.php" -mtime -7

# Find files with dangerous permissions
find . -name "*.php" -perm 0777

# Check for base64_decode usage
grep -r "base64_decode" --include="*.php" .
```