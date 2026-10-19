# Project-4

# Mediroza General Hospital — Penetration Test

Authorised penetration testing project against a training target (`medirozahospital.com`), completed as part of the Networkwalks pentest training program (Batch B082, Week 4).

> ⚠️ **Educational use only.** This repo documents a training exercise against a target explicitly authorised for testing. None of the techniques here should be used against any system without explicit written permission from its owner.

---

## 🎯 Objective

Identify vulnerabilities in the hospital's externally facing patient portal that an unauthenticated attacker could exploit, and trace how individual weaknesses chain together to escalate access and data exposure.

## 🧭 Scope

- Target: `https://medirozahospital.com`
- Public-facing web application and its publicly accessible subdirectories
- Black-box approach — no credentials provided at the start

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| `curl` | Fetching `robots.txt` and basic recon |
| Browser DevTools | Manual exploration of login and portal pages |
| Networkwalks Hash Calculator | Extracting PDF password hashes |
| Networkwalks Password Cracker | Cracking PDF passwords against wordlists |
| `exiftool` | Reading hidden file metadata |
| Claude | Converting raw SQL dump data into readable tables |

## 🪜 Methodology

1. **Reconnaissance** — enumerate hidden directories via `robots.txt` and manual browsing
2. **Initial Access** — test the login page for information disclosure and injection flaws, then exploit them
3. **Post-Exploitation Analysis** — extract and crack protected files obtained after gaining access; analyse metadata for further clues
4. **Deep Reconnaissance** — follow clues to locate and assess additional exposed assets

## 🔍 Attack Chain

```
robots.txt → /patient/ and /old/ disclosed
        ↓
Username enumeration → "admin" confirmed valid
        ↓
SQL injection (admin' --) → authentication bypassed
        ↓
3 patient report PDFs downloaded → weak passwords cracked via wordlist
        ↓
PDF metadata (Author: j.malik) → note reveals /old/ backup location
        ↓
/old/ → directory listing enabled → mediroza_db_backup_2019.sql exposed
        ↓
Staff + shareholder data recovered in plain text → j.malik identified as
Jameel Malik, IT Systems Administrator
```

## 📋 Findings Summary

| # | Vulnerability | Location | Risk |
|---|---|---|---|
| 1 | Username enumeration on login page | `/patient/login.php` | Medium |
| 2 | SQL injection login bypass | `/patient/login.php` | Critical |
| 3 | Encrypted PDFs accessible after login bypass | `/patient/reports/` | High |
| 4 | Weak PDF passwords crackable with a wordlist | `patient_report_*.pdf` | High |
| 5 | Sensitive metadata left in patient PDF files | `patient_report_3.pdf` | Medium |
| 6 | Forgotten backup folder with directory listing enabled | `/old/` | Critical |
| 7 | Confidential staff salaries and shareholder data exposed in plain text | `/old/mediroza_db_backup_2019.sql` | Critical |

## ✅ Key Recommendations

- Use **parameterised queries / prepared statements** — never build SQL from raw user input
- Return **identical login error messages** regardless of which field was wrong
- Store PDFs and backups **outside the public web root**, with server-side authorisation checks
- **Strip metadata** from files before distribution (`exiftool -all= filename.pdf`)
- **Disable directory listing** site-wide and remove stale backup files from public folders
- Use **strong, unique passwords** for any password-protected files, and encrypt sensitive backups at rest


## ⚠️ Disclaimer

This project was conducted for educational purposes only, against a target explicitly authorised for security testing by Networkwalks. These techniques must never be applied to any system without explicit written permission from the owner.
