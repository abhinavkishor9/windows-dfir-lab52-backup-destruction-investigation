# Troubleshooting Notes — Lab 52 Backup Destruction Investigation

## 1. No VSS Shadow Copies Found

### Problem

Running:

`vssadmin list shadows`

returned:

`No items found that satisfy the query.`

### Explanation

The test system did not have any Volume Shadow Copies available at the time of investigation.

### DFIR Interpretation

This is valid baseline evidence.

It does not mean that VSS is broken, and it does not indicate that an attacker deleted the shadow copies.

The important distinction is:

`No shadow copies present`

does not automatically mean:

`Shadow copies were maliciously deleted`

---

## 2. No VSS Shadow Storage Found

### Problem

Running:

`vssadmin list shadowstorage`

returned:

`No items found that satisfy the query.`

### Explanation

No VSS shadow-copy storage was configured on the system.

### DFIR Interpretation

This establishes that there was no existing shadow-storage configuration to investigate as part of the controlled lab.

---

## 3. No Windows Backup Versions

### Problem

`wbadmin get versions` did not provide an existing backup history.

### Explanation

The test system may simply not have Windows Backup versions configured.

### DFIR Interpretation

Do not interpret the absence of backup versions as evidence that an attacker destroyed backups.

A baseline must be established before claiming that something was removed.

---

## 4. PowerShell Script Execution Blocked

### Problem

The initial command:

`powershell.exe -File "C:\BackupDestructionLab\backup-destruction.ps1"`

failed with a SecurityError indicating that running scripts was disabled.

### Cause

The system's PowerShell Execution Policy prevented normal `.ps1` script execution.

### Resolution

The controlled script was executed using:

`powershell.exe -ExecutionPolicy Bypass -File "C:\BackupDestructionLab\backup-destruction.ps1"`

This bypass applied only to the PowerShell process used for the lab.

### DFIR Significance

The execution-policy bypass is itself important investigative evidence.

If command-line logging is enabled, an analyst may be able to identify the use of:

`-ExecutionPolicy Bypass`

through process creation telemetry.

---

## 5. Do Not Permanently Disable Execution Policy

### Problem

It may be tempting to modify the system-wide Execution Policy to allow the script to run.

### Recommended Approach

Do not weaken the machine's permanent security configuration just for this lab.

Use a process-level bypass:

`powershell.exe -ExecutionPolicy Bypass -File "C:\BackupDestructionLab\backup-destruction.ps1"`

### DFIR Principle

Controlled testing should make the smallest possible change to the environment.

---

## 6. PowerShell Event ID 4104 Not Available

### Problem

Event ID 4104 may not appear.

### Possible Causes

- Script Block Logging is disabled.
- The PowerShell log does not contain the relevant timeframe.
- Events were overwritten.
- Logging configuration differs from the expected environment.

### Check

Use Event Viewer:

`Applications and Services Logs > Microsoft > Windows > PowerShell > Operational`

Or PowerShell:

`Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-PowerShell/Operational"; Id=4104} -MaxEvents 50`

### DFIR Note

If 4104 is absent, do not conclude that PowerShell was not executed.

Use other available evidence such as Event 4688 and Sysmon Event ID 1 if available.

---

## 7. Security Event ID 4688 Not Available

### Problem

Event ID 4688 cannot be found.

### Possible Causes

- Process Creation auditing is disabled.
- The Security log does not contain the relevant timeframe.
- Events were overwritten.
- Insufficient privileges were used to query the Security log.

### Check

`Get-WinEvent -FilterHashtable @{LogName="Security"; Id=4688} -MaxEvents 50`

### DFIR Note

The absence of 4688 reduces process-level visibility but does not prove that no process executed.

---

## 8. Sysmon Event ID 3 Not Available

### Problem

No Sysmon Event ID 3 events are found.

### Possible Causes

- Network connection monitoring is not enabled by the Sysmon configuration.
- No qualifying network connection occurred.
- The relevant event is outside the queried timeframe.
- Events were overwritten.

### Check

`Get-WinEvent -FilterHashtable @{LogName="Microsoft-Windows-Sysmon/Operational"; Id=3} -MaxEvents 50`

### DFIR Note

No Event ID 3 does not prove that no network activity occurred.

---

## 9. Network Event Appears During the Investigation

### Problem

Sysmon Event ID 3 shows a connection around the time of the PowerShell execution.

### Investigation Approach

Record:

- Timestamp
- Process
- Process ID
- Source IP
- Destination IP
- Destination port
- Protocol

Then determine whether the connection is expected.

### Important

`Network connection ≠ Backup exfiltration`

Additional evidence is required to establish that backup data was transmitted.

---

## 10. Script Executes but Expected Telemetry Is Missing

### Problem

The script executes successfully, but one or more expected events do not appear.

### Resolution

Do not immediately repeat the activity multiple times.

First determine:

- Which logging sources are enabled.
- Whether the event existed before the test.
- Whether the event was generated after execution.
- Whether the event channel contains the correct timeframe.
- Whether Sysmon or audit policy configuration filters the event.

### DFIR Principle

Avoid generating unnecessary additional activity when the original evidence is still available.

---

## 11. VSS Destruction Command Consideration

### Problem

The investigation concerns commands such as:

`vssadmin delete shadows`

### Safe-Lab Approach

Do not execute destructive VSS commands against the real Windows installation merely to generate evidence.

The purpose of this lab is to investigate backup-destruction behavior safely.

### DFIR Principle

A lab should reproduce the investigative concept without unnecessarily damaging the host.

---

## 12. Confusing Simulation With Actual Backup Destruction

### Problem

A PowerShell script named `backup-destruction.ps1` was executed, and the analyst may conclude that backups were destroyed.

### Correct Interpretation

The name of the script does not establish what actually happened.

Evidence must show:

- What commands were executed.
- Whether the commands succeeded.
- What recovery resources existed before execution.
- What recovery resources existed afterward.

In this lab, the initial VSS state showed no shadow copies or shadow storage.

Therefore, the investigation should not claim that real VSS recovery data was deleted.

---

## Final Troubleshooting Lesson

The major troubleshooting lesson from Lab 52 was that security controls can affect the evidence-generation process itself. PowerShell Execution Policy initially prevented the controlled script from running, requiring a process-level bypass.

At the same time, the absence of VSS recovery data meant that a real destructive operation could not safely be demonstrated.

Both situations reinforce an important DFIR principle:

`Record the environment first, understand the limitation, and never manufacture evidence by unnecessarily weakening or damaging the system.`
