
## 👤 Author

**LinkState** — v2.0 — April 2026

# 🛡️ Veeam Backup Validator

> A PowerShell automation script to validate Veeam backups and optionally send HTML reports via email. Designed for **Veeam Backup & Replication v12 and v13**, schedulable via Windows Task Scheduler.

![PowerShell](https://img.shields.io/badge/PowerShell-5.1%2B-blue?logo=powershell)
![Veeam](https://img.shields.io/badge/Veeam-v12%20%7C%20v13-00B336?logo=veeam)
![Windows](https://img.shields.io/badge/Platform-Windows-0078D6?logo=windows)
![Version](https://img.shields.io/badge/Version-2.0-informational)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 📋 Overview

`veeam-BackupValidator.ps1` automates the validation of Veeam backup jobs using `Veeam.Backup.Validator.exe`. For each backup job (or a specific server within a job), it:

- Runs the Veeam native validator and captures the exit code
- Generates an HTML report per backup
- Merges all reports into a single consolidated HTML file
- Optionally sends the report via email to a service desk or operations team
- Rotates and cleans up old log/report files automatically (30-day retention)

---

## ✅ Requirements

| Requirement | Details |
|---|---|
| OS | Windows Server 2016 / 2019 / 2022 |
| PowerShell | 7 or later |
| Veeam B&R | v12 or v13 |
| Veeam PowerShell Snap-in | Must be installed and importable |
| `Veeam.Backup.Validator.exe` | Present in Veeam installation directory |
| SMTP relay | Required only if email notifications are enabled |

> **Note:** The script must be run on the **Veeam Backup Server** or a machine with the Veeam console and PowerShell snap-in installed.

---

## ⚙️ Configuration

Before using the script, edit the following variables at the top of the file:

```powershell
$customer        = "YOUR CUSTOMER"                        # Customer/environment label
$email_infoTO    = "your-servicedesk@your-domain"         # Recipient email
$email_infoFrom  = "$($env:COMPUTERNAME)@your-domain"     # Sender email
$email_infoServer = "YOUR smart-relay"                    # SMTP relay hostname
```

---

## 🚀 Usage

```powershell
.\veeam-BackupValidator.ps1 [<jobName>] [<serverName>] [-noTicket]
```

### Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `jobName` | String | `*` | Name of the Veeam backup job to validate. Use `*` for all jobs. |
| `serverName` | String | `*` | Name of a specific server within the job. Use `*` for all servers. |
| `-noTicket` | Switch | `$true` | When present, suppresses email notification to the service desk. |

### Examples

```powershell
# Validate all backup jobs (no email)
.\veeam-BackupValidator.ps1

# Validate a specific job for all its servers (no email)
.\veeam-BackupValidator.ps1 "D01-JOB-NAME"

# Validate a specific server within a job (no email)
.\veeam-BackupValidator.ps1 "D01-JOB-NAME" "YOUR-SERVER" -noTicket

# Validate and send email report to service desk
.\veeam-BackupValidator.ps1 "D01-JOB-NAME" "YOUR-SERVER"
```

> **Tip:** Omit `-noTicket` (or explicitly set it to `$false`) to enable email delivery of the HTML report.

---

## 📁 Output Files

All output files are saved in the same directory as the script:

| File | Description |
|---|---|
| `VeeamValidation_<timestamp>.html` | Consolidated HTML validation report |
| `VeeamValidation_<timestamp>.log` | Plain-text execution log |
| `validationResult\Validation_*.html` | Temporary per-backup HTML files (auto-deleted after merge) |

Files older than **30 days** are automatically purged on each run.

---

## 🗓️ Scheduling with Windows Task Scheduler

You can automate the validator using **Windows Task Scheduler**:

1. Open **Task Scheduler** and create a new task.
2. Set the **Trigger** (e.g., daily at 06:00 AM after backup jobs complete).
3. Set the **Action** to:
   - **Program:** `powershell.exe`
   - **Arguments:**
     ```
     -NonInteractive -ExecutionPolicy Bypass -File "C:\Scripts\veeam-BackupValidator.ps1" "YOUR-JOB" "YOUR-SERVER"
     ```
4. Run the task as a service account with **Veeam Operator** (or higher) permissions.
5. Enable **"Run whether user is logged on or not"**.

> **Important:** Ensure the executing account has rights to run Veeam PowerShell cmdlets and access `Veeam.Backup.Validator.exe`.


<img width="635" height="482" alt="image" src="https://github.com/user-attachments/assets/85c424f8-49b4-4d31-95e6-fc8b668fd8f0" />

<img width="631" height="481" alt="image" src="https://github.com/user-attachments/assets/b68f4115-20e2-43da-9dc4-72596455ff9f" />

Program/script: "C:\Program Files\PowerShell\7\pwsh.exe"
Add arguments: -file "C:\Your-dir\BackupValidator\veeam-BackupValidator.ps1"

<img width="646" height="673" alt="image" src="https://github.com/user-attachments/assets/617696c0-179c-4277-aa23-c04a2b5d9392" />


---

## 📊 Log Output Example

```
[INFO][06:00:01]: 04-08-2026_0600
[INFO][06:00:01]: Scan for Job List...
[INFO][06:00:01]: Job Name: D01-JOB-NAME
[INFO][06:00:01]: Server Name: YOUR-SERVER
[INFO][06:00:02]: Validating     : D01-JOB-NAME-YOUR-SERVER
[POSITIVE][06:00:45]: Validation Result -> OK
[INFO][06:00:45]: Global Validation Statistics: 1 : 1 : 0 (tot:ok:ko)
[POSITIVE][06:00:46]: Mail Sent -> OK
[INFO][06:00:46]: Delete temp files
[INFO][06:00:46]: File Removed
```

---

## 🔄 How It Works

```
Start
  │
  ├─► Get-VBRBackup (all or filtered by jobName)
  │     └─► FindChildBackups() → iterate sub-children
  │           └─► Filter by serverName (if specified)
  │                 └─► Run Veeam.Backup.Validator.exe
  │                       ├─► Exit 0  → OK
  │                       └─► Exit ≠0 → ERROR
  │
  ├─► Merge per-backup HTML reports → single OutputFile
  ├─► Send email (if -noTicket is $false)
  └─► Cleanup temp files + rotate old logs (>30 days)
```

---

## 🤝 Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you'd like to change.

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---





