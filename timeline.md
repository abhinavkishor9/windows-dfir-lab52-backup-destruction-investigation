# Timeline — Lab 52 Backup Destruction Investigation

## Timeline Purpose

This timeline documents the sequence of actions performed during the controlled backup-destruction investigation and the telemetry obtained from the Windows system.

Actual timestamps from Event Viewer should be inserted where available.

## Investigation Timeline

| Order | Source | Activity | Significance |
|---:|---|---|---|
| 1 | File System | `C:\BackupDestructionLab` created | Establishes controlled investigation workspace |
| 2 | File System | Simulated backup files created | Provides harmless test artifacts |
| 3 | File System | Baseline file metadata collected | Establishes initial state |
| 4 | VSSAdmin | `vssadmin list shadows` executed | Checks for existing shadow copies |
| 5 | VSSAdmin | No shadow copies found | Establishes VSS baseline |
| 6 | VSSAdmin | `vssadmin list shadowstorage` executed | Checks VSS storage configuration |
| 7 | VSSAdmin | No shadow storage found | Establishes recovery-storage baseline |
| 8 | Windows Backup | `wbadmin get versions` executed | Checks for available Windows Backup versions |
| 9 | PowerShell | Initial script execution attempted | Tests normal PowerShell execution |
| 10 | PowerShell | Execution Policy blocked script | Documents host security control |
| 11 | PowerShell | Process-level `-ExecutionPolicy Bypass` used | Executes controlled lab script without permanently changing policy |
| 12 | Event Viewer | PowerShell Event ID 4104 observed | Provides PowerShell script-level telemetry |
| 13 | Event Viewer | Security Event ID 4688 observed | Provides process creation evidence |
| 14 | Event Viewer | Sysmon Event ID 3 observed | Provides network connection evidence |
| 15 | DFIR Analysis | Events correlated by timestamp | Connects activity across telemetry sources |
| 16 | VSSAdmin | Recovery state reviewed again | Confirms controlled nature of investigation |
| 17 | File System | Simulated backup directory reviewed | Confirms state of controlled artifacts |
| 18 | DFIR Analysis | Final assessment completed | Determines what the evidence actually supports |

## Confirmed Telemetry

The following telemetry was observed during the investigation:

| Source | Event ID | Evidence Type |
|---|---:|---|
| PowerShell | 4104 | Script Block Logging |
| Windows Security | 4688 | Process Creation |
| Sysmon | 3 | Network Connection |

## VSS Baseline

The initial VSS investigation produced:

`vssadmin list shadows`

Result:

`No items found that satisfy the query.`

The initial VSS shadow-storage check produced:

`vssadmin list shadowstorage`

Result:

`No items found that satisfy the query.`

This means that no existing VSS recovery points or configured shadow-copy storage were available during the baseline.

## PowerShell Execution Timeline

The controlled PowerShell script was initially executed without an Execution Policy override.

Windows prevented execution because script execution was disabled by the configured policy.

The controlled script was then executed using:

`powershell.exe -ExecutionPolicy Bypass -File "C:\BackupDestructionLab\backup-destruction.ps1"`

This generated the activity that was subsequently investigated.

## Event Correlation

The relevant evidence chain can be represented as:

`PowerShell Script Execution`

`↓`

`Event ID 4104`

`↓`

`Event ID 4688`

`↓`

`Sysmon Event ID 3`

`↓`

`Timestamp Correlation`

`↓`

`DFIR Assessment`

The exact relationship between events should be confirmed using their timestamps, process identifiers, command lines, and other available fields.

## Evidence Interpretation

PowerShell Event ID 4104 provides script-level information.

Security Event ID 4688 provides process creation information.

Sysmon Event ID 3 provides network connection information.

Together, these events provide useful endpoint context around the controlled activity.

However, they do not independently prove that a backup or recovery mechanism was destroyed.

## Evidence Gaps

The investigation should document the following limitations:

- No VSS shadow copies existed at baseline.
- No VSS shadow storage was configured at baseline.
- The controlled environment did not contain real backup infrastructure that could safely be destroyed.
- A process execution event does not prove that a destructive command succeeded.
- A PowerShell event does not automatically establish malicious intent.
- A network connection does not automatically establish data exfiltration.

## Final Timeline Assessment

The investigation established a controlled sequence involving PowerShell execution, process creation, and network telemetry. The initial VSS checks provided important baseline evidence showing that no shadow copies or shadow storage were available before the simulation.

The execution-policy restriction and subsequent process-level bypass were also significant parts of the investigation because they demonstrate how host security controls can affect both execution and forensic visibility.

The available evidence supports the investigation of simulated recovery-inhibiting behavior but does not establish that real Windows recovery infrastructure was destroyed.

## Analyst Note

Replace the logical order shown in this document with the actual timestamps collected from Event Viewer when finalizing the investigation report.

Do not invent timestamps.

The final timeline should preserve the original evidence time and clearly distinguish between observed activity, analyst interpretation, and assumptions.
