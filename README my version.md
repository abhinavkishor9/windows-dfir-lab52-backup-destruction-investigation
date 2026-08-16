# windows-dfir-lab52-backup-destruction-investigation
## Overview

Backup destruction is the deliberate removal, modification, or disabling of backup and recovery mechanisms so that an organization cannot easily restore systems or data after an attack.

Attackers may target:

Volume Shadow Copies
Windows recovery points
Backup catalogs
Backup files
Backup services
Recovery configuration
Cloud or network backup repositories

The key idea is:

Normal backup
     ↓
Recovery capability
     ↓
Incident occurs
     ↓
System can be restored

An attacker may instead attempt:

Compromise
    ↓
Privilege escalation
    ↓
Identify backups
    ↓
Delete/disable recovery mechanisms
    ↓
Encrypt/delete data
    ↓
Recovery becomes difficult

This is particularly important during ransomware investigations.


This investigation began by establishing the baseline state of Volume Shadow Copy Service (VSS) recovery data and Windows Backup information. A controlled PowerShell-based simulation was then executed to generate investigation telemetry. PowerShell Script Block Logging, Windows Security process creation telemetry, and Sysmon network telemetry were reviewed to reconstruct the activity.

## Lab Objectives

- Establish the system's recovery baseline before analyzing any suspicious activity.
- Determine whether VSS recovery resources are present and document their initial state.
- Identify the processes involved in the controlled backup-destruction simulation.
- Examine PowerShell activity to understand what script execution occurred during the investigation.
- Correlate process creation and PowerShell telemetry to build a reliable sequence of events.
- Review network activity around the execution timeframe and determine whether any related communication occurred.
- Identify the effect of PowerShell Execution Policy on the investigation and document how the controlled execution was performed.
- Differentiate attempted recovery-inhibiting activity from successful backup destruction rather than assuming that execution equals impact.
- Assess the available evidence for signs of suspicious behavior, including execution context, timing, user, parent process, and command line.
- Document evidence gaps and limitations so that conclusions remain based on what the telemetry actually proves.

## Investigation Scenario

A Windows workstation shows activity that may indicate an attempt to interfere with backup or recovery mechanisms. The SOC analyst is asked to determine whether the activity represents legitimate administration, a failed attempt, or potentially malicious recovery-inhibiting behavior.

The investigation focuses on reconstructing the activity from endpoint evidence rather than assuming that a suspicious command automatically resulted in backup destruction.

A PowerShell script is executed from a controlled lab directory.
The system initially blocks the script because of its Execution Policy.
The script is later executed using a process-level Execution Policy bypass.
Windows Security Event ID 4688 records process creation activity.
PowerShell Event ID 4104 provides script execution telemetry.
Sysmon Event ID 3 provides network connection telemetry around the activity.
VSS inspection shows that no shadow copies were available at the beginning of the investigation.
The analyst must correlate these events and determine what actually happened, what can be proven, and what remains unknown.

## Lab Environment

- Operating System: Windows
- Investigation Type: Windows DFIR
- Primary Tools:
  - PowerShell
  - Event Viewer
  - Sysmon
  - VSSAdmin
  - WBAdmin
- Controlled Lab Directory:

`C:\BackupDestructionLab`

## Evidence Sources

### VSSAdmin

VSSAdmin was used to establish the baseline state of Volume Shadow Copies and shadow-copy storage.

Commands used included:

`vssadmin list shadows`

`vssadmin list shadowstorage`

The investigation returned no existing shadow copies and no configured shadow-copy storage.

### Windows Backup

`wbadmin get versions` was used to check for available Windows Backup versions.

### Windows Security Event ID 4688

Security Event ID 4688 was used to investigate process creation associated with the controlled activity.

### PowerShell Event ID 4104

PowerShell Script Block Logging Event ID 4104 was used to examine PowerShell script activity.

### Sysmon Event ID 3

Sysmon Event ID 3 was reviewed for network connection activity occurring around the investigation timeframe.

## Investigation Workflow

1. Created a controlled investigation directory.
2. Created simulated backup files.
3. Collected baseline file information.
4. Checked the VSS shadow-copy state.
5. Checked VSS shadow storage.
6. Checked Windows Backup information.
7. Reviewed relevant Windows logging.
8. Prepared the controlled PowerShell simulation.
9. Attempted normal script execution.
10. Encountered a PowerShell Execution Policy restriction.
11. Used a process-level `-ExecutionPolicy Bypass` to execute the controlled script.
12. Recorded the execution timeframe.
13. Investigated PowerShell Event ID 4104.
14. Investigated Windows Security Event ID 4688.
15. Investigated Sysmon Event ID 3.
16. Correlated the available telemetry.
17. Verified that real recovery infrastructure had not intentionally been destroyed.
18. Documented the investigation findings and limitations.

## Initial VSS Findings

The following commands were executed before the simulation:

`vssadmin list shadows`

`vssadmin list shadowstorage`

The system returned:

`No items found that satisfy the query.`

This established that no existing VSS shadow copies or shadow-copy storage were available on the test system at the time of the baseline collection.

This finding is important because the investigation did not begin with an existing set of VSS recovery points that could be safely removed.

## PowerShell Execution Policy Finding

When the controlled PowerShell script was initially executed normally, Windows prevented the script from running because PowerShell script execution was restricted by the system's Execution Policy.

The controlled lab was therefore executed using a process-level bypass:

`powershell.exe -ExecutionPolicy Bypass -File "C:\BackupDestructionLab\backup-destruction.ps1"`

The Execution Policy was not permanently changed.

This was documented as part of the investigation because the command-line execution method itself becomes relevant endpoint evidence.

## Observed Telemetry

The following telemetry was observed during the investigation:

| Source | Event ID | Purpose |
|---|---:|---|
| PowerShell | 4104 | Script Block Logging |
| Windows Security | 4688 | Process creation |
| Sysmon | 3 | Network connection |

These events were used to reconstruct the controlled activity and correlate the execution timeframe.



## MITRE ATT&CK Relevance

The primary ATT&CK technique associated with this investigation is:

**T1490 — Inhibit System Recovery**

T1490 describes activity intended to prevent or interfere with system recovery.

PowerShell activity may also be relevant to:

**T1059.001 — PowerShell**

However, ATT&CK mapping should be based on the actual evidence observed during an investigation rather than on the presence of a particular command or event alone.


## Disclaimer

This investigation was performed in a controlled environment using simulated backup-destruction activity. No real organizational backup data or confidential information was used, and real recovery infrastructure was not intentionally destroyed.
