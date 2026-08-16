# Investigation Notes 

## Investigation Environment

Controlled investigation directory:

`C:\BackupDestructionLab`

The directory was used to store simulated backup material and the PowerShell script used during the controlled investigation.

## Baseline Collection

Before executing the simulation, the existing VSS state was checked.

### VSS Shadow Copies

Command:

`vssadmin list shadows`

Result:

`No items found that satisfy the query.`

This indicated that no VSS shadow copies were present at the time of baseline collection.

### VSS Shadow Storage

Command:

`vssadmin list shadowstorage`

Result:

`No items found that satisfy the query.`

This indicated that no VSS shadow-copy storage was configured on the test system.

## Windows Backup Check

Windows Backup information was checked using:

`wbadmin get versions`

The purpose was to determine whether Windows Backup versions were available on the system.

The result was treated as baseline evidence rather than as evidence of an attack.

## Simulated Backup Data

Harmless files were created inside the controlled investigation directory.

Examples included:

`backup-data.txt`

`user-config.bak`

`application-data.bak`

These files represented simulated backup material.

No real organizational backup files were used.

## Controlled PowerShell Simulation

A PowerShell script named:

`backup-destruction.ps1`

was created inside the investigation directory.

The script was intended to simulate backup-destruction behavior for telemetry generation without intentionally destroying real Windows recovery data.

## Execution Policy Issue

The first attempt to execute the script normally resulted in a PowerShell SecurityError because script execution was disabled by the system's Execution Policy.

The error demonstrated that the system's current PowerShell configuration prevented direct execution of the `.ps1` file.

The Execution Policy was not permanently modified.

## Process-Level Bypass

The script was subsequently executed using:

`powershell.exe -ExecutionPolicy Bypass -File "C:\BackupDestructionLab\backup-destruction.ps1"`

This bypass applied to the PowerShell process used for the controlled lab rather than changing the system's permanent execution policy.

This execution method is itself important evidence because the command line can potentially be captured by process-creation telemetry.

## PowerShell Event ID 4104

PowerShell Event ID 4104 was observed through Event Viewer.

The event was used to investigate script execution and determine what PowerShell activity was recorded during the controlled simulation.

Relevant information includes:

- Timestamp
- Script content
- PowerShell execution context
- Script path or associated content
- User context when available

Event ID 4104 provides useful script-level context when Script Block Logging is enabled.

## Windows Security Event ID 4688

Security Event ID 4688 was observed.

This event was used to investigate process creation around the execution timeframe.

Relevant fields include:

- New Process Name
- Creator Process Name
- Process ID
- Command Line
- Subject User
- Timestamp

The event can help establish which process launched PowerShell and under which account.

## Sysmon Event ID 3

Sysmon Event ID 3 was observed.

This event was reviewed for network connection activity around the controlled execution.

Relevant fields include:

- Timestamp
- Process ID
- Process Image
- Source IP
- Destination IP
- Destination Port
- Protocol

The network event was treated as supporting evidence only.

A network connection does not independently demonstrate that backup data was transmitted.

## Event Correlation

The investigation correlated the available telemetry using timestamps.

The analytical sequence was:

`PowerShell Execution`

`↓`

`Security Event 4688`

`↓`

`PowerShell Event 4104`

`↓`

`Sysmon Event 3`

`↓`

`Timeline Correlation`

The objective was to determine whether the different telemetry sources represented the same controlled activity.

## Recovery State Assessment

The initial VSS checks demonstrated:

- No shadow copies were present.
- No shadow-copy storage was configured.

Therefore, there was no existing VSS recovery data that could be safely removed as part of the lab.

The investigation deliberately avoided executing destructive commands against real recovery infrastructure.

## Evidence Interpretation

The observed events demonstrate that the controlled PowerShell activity generated useful endpoint telemetry.

Event 4104 provides script-level evidence.

Event 4688 provides process-creation evidence.

Sysmon Event 3 provides network-connection evidence.

Together, these sources allow an analyst to reconstruct the execution context.

However, none of these events alone proves that a backup was destroyed.

