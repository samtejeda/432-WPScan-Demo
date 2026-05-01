# WPScan — Hands-On Demo

---

## What is WPScan?

WPScan is a black-box WordPress security scanner used by penetration testers and site owners to find vulnerabilities in WordPress installations. It comes pre-installed on Kali Linux and can discover:

- WordPress version and known vulnerabilities
- Installed plugins and themes with version numbers
- Valid usernames on the site
- Exposed sensitive files like config backups
- Weak passwords via brute force attacks

> ⚠️ **Important:** Only use WPScan on sites you own or have explicit written permission to test. Unauthorized scanning is illegal.

---

## One-Time Setup — Create Demo Wordlist

Run this once on your Kali VM before starting the demo:

```bash
cat > ~/demo-passwords.txt << 'WORDLIST'
123456
admin
letmein
password
qwerty
wordpress
hello123
password123
abc123
welcome
WORDLIST
```
Or alternatively you can use the built-in Kali VM wordlist, use this command to use unzip it:
```bash
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
```
And replace
``` bash
~/demo-passwords.txt
```
With
``` bash
/usr/share/wordlists/rockyou.txt
```
In the commands that use the worlist (6 and 7 in this demo).

---

## Demo Commands
⚠️ Make sure to replace 'https://target-wp-site.com' with the actual target WordPress site URL.

### 1. Update WPScan Database

Always run this first to make sure WPScan has the latest vulnerability data.

```bash
wpscan --update
```

**What to look for:** Should say `Database updated` or `Already up to date`.

---

### 2. Basic Scan

A general scan that fingerprints the WordPress installation — version, server software, interesting files.

```bash
wpscan --url https://target-wp-site.com
```

**What to look for:** WordPress version, readme.html, XML-RPC status, robots.txt entries.

---

### 3. User Enumeration

Discovers valid WordPress usernames using multiple techniques — author archives, REST API, RSS feed, and ID brute forcing.

```bash
wpscan --url https://target-wp-site.com -e u
```

**What to look for:** All 4 usernames discovered. Note how WPScan shows which technique found each one.

---

### 4. Sensitive File Detection

Checks for exposed configuration backups and database exports that should never be publicly accessible.

```bash
wpscan --url https://target-wp-site.com -e cb,dbe
```

**What to look for:** `wp-config.txt` flagged as a critical finding — this file contains database credentials.

---

### 5. Full Enumeration

Runs everything at once — plugins, themes, users, and sensitive files in a single scan.

```bash
wpscan --url https://target-wp-site.com \
  -e ap,at,u,cb,dbe \
  --plugins-detection mixed
```

**What to look for:** A comprehensive report covering all attack surfaces.

---

### 6. Password Brute Force

Uses the wordlist you created to try cracking admin_user's password. WPScan tries each password against the WordPress login page.

```bash
wpscan --url https://target-wp-site.com \
  -U admin_user \
  -P ~/demo-passwords.txt \
  --password-attack wp-login
```

**What to look for:** A `[SUCCESS]` line showing the cracked username and password.

---

### 7. XML-RPC Brute Force

An alternative brute force method using the XML-RPC API. It can send multiple password guesses per request, making it faster than the login method.

```bash
wpscan --url https://target-wp-site.com \
  -U admin_user \
  -P ~/demo-passwords.txt \
  --password-attack xmlrpc
```

**What to look for:** Same result as command 6 but uses a different attack vector.

---

### 8. Save Output to File

Exports scan results to a text file — essential for real penetration testing reports.

```bash
wpscan --url https://target-wp-site.com -e u \
  --output ~/wpscan-report.txt \
  --format cli-no-colour
```

**What to look for:** File saved at `~/wpscan-report.txt`. Open it with:

```bash
cat ~/wpscan-report.txt
```

---

## Example of possible findings (may vary depending on the target site)

| Finding | Type | Details |
|---|---|---|
| WordPress 6.9.4 | Version | Detected from RSS feed and readme.html |
| Contact Form 7 | Plugin | Version fingerprinted from asset URLs |
| WooCommerce | Plugin | Version detected with 100% confidence |
| Yoast SEO | Plugin | Found in HTML comments |
| WP Statistics | Plugin | Version from readme.txt |
| Akismet | Plugin | Inactive — still detectable |
| Astra | Theme | Active theme, version detected |
| OceanWP | Theme | Inactive, still found |
| admin_user | User | Via author archive + REST API + RSS |
| editor_jane | User | Via author archive + REST API |
| john_contrib | User | Via author archive |
| subscriber_bob | User | Via ID brute forcing |
| wp-config.txt | Sensitive File | Critical — contains DB credentials |
| admin_user:password123 | Cracked Credential | Found via brute force |

---

## Legal & Ethical Reminder

All scanning in this demo should be performed on a site that you own or have permission to scan. Never run WPScan or any security tool against a website without explicit written permission from the owner. Unauthorized scanning may be illegal under the Computer Fraud and Abuse Act (CFAA) and equivalent laws.
