# Windows DFIR Lab 52 — Backup Destruction Investigation

## Overview

This lab investigates backup destruction and recovery-inhibiting behavior from a Windows DFIR perspective. The investigation was performed in a controlled environment using simulated backup data rather than deleting real Windows recovery data. The purpose was to understand how an analyst can identify and correlate activity that may interfere with backup or recovery mechanisms.

The investigation began by establishing the baseline state of Volume Shadow Copy Service (VSS) recovery data and Windows Backup information. A controlled PowerShell-based simulation was then executed to generate investigation telemetry. PowerShell Script Block Logging, Windows Security process creation telemetry, and Sysmon network telemetry were reviewed to reconstruct the activity.

## Lab Objectives

- Understand how attackers may attempt to inhibit system recovery.
- Establish a baseline for Windows recovery mechanisms before an investigation.
- Examine the availability of Volume Shadow Copies and shadow storage.
- Investigate controlled backup-destruction simulation activity.
- Analyze PowerShell Script Block Logging.
- Analyze Windows Security Event ID 4688.
- Analyze Sysmon Event ID 3.
- Correlate timestamps across multiple Windows telemetry sources.
- Investigate the effect of PowerShell Execution Policy on the controlled simulation.
- Distinguish simulated backup-destruction activity from actual recovery-system modification.
- Document evidence limitations and avoid unsupported conclusions.

## Investigation Scenario

A Windows workstation is suspected of running activity intended to interfere with backup and recovery mechanisms. Such behavior can be associated with ransomware and other destructive attacks because removing recovery options can make restoration more difficult.

A controlled simulation was performed using a PowerShell script inside a dedicated investigation directory. Before the simulation, the existing VSS state was examined and no shadow copies or shadow storage were found. The script was then executed for investigation purposes without intentionally modifying real Windows recovery infrastructure.

The resulting PowerShell, process-creation, and network telemetry was examined to determine what could be established from the available evidence.

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

## Key Findings

- No VSS shadow copies were present during the initial baseline.
- No VSS shadow-copy storage was configured during the initial baseline.
- A controlled backup-destruction simulation was performed.
- Normal PowerShell script execution was initially blocked by Execution Policy.
- The controlled script was executed using a process-level `-ExecutionPolicy Bypass`.
- PowerShell Event ID 4104 was available for script activity investigation.
- Windows Security Event ID 4688 was available for process creation analysis.
- Sysmon Event ID 3 was available for network activity analysis.
- The available telemetry allowed the controlled execution to be investigated from multiple sources.
- The simulation did not intentionally delete real VSS recovery data.
- Observing PowerShell, process creation, or network events does not independently prove malicious backup destruction.

## DFIR Interpretation

Backup destruction is significant because attackers may remove recovery mechanisms before encrypting or destroying data. However, a command or process associated with backup management is not automatically malicious. An analyst must establish the executing user, command line, parent process, execution time, target recovery mechanism, success or failure of the operation, and subsequent activity.

In this controlled lab, the available telemetry demonstrated how destructive-looking activity can be investigated without actually destroying the system's recovery infrastructure.

## MITRE ATT&CK Relevance

The primary ATT&CK technique associated with this investigation is:

**T1490 — Inhibit System Recovery**

T1490 describes activity intended to prevent or interfere with system recovery.

PowerShell activity may also be relevant to:

**T1059.001 — PowerShell**

However, ATT&CK mapping should be based on the actual evidence observed during an investigation rather than on the presence of a particular command or event alone.

## Evidence Limitations

The test system did not have existing VSS shadow copies or configured shadow-copy storage when the baseline was collected.

Therefore, the lab could not safely demonstrate the deletion of an existing real shadow copy.

The investigation instead focused on controlled simulation and telemetry correlation.

Event availability is also dependent on Windows and Sysmon configuration. The absence of a particular event should therefore be treated as a telemetry limitation rather than automatic proof that an activity did not occur.

## Conclusion

This investigation demonstrated how backup-destruction behavior can be investigated through multiple Windows telemetry sources without intentionally damaging recovery infrastructure. VSS inspection established the initial recovery baseline, while PowerShell Event ID 4104, Windows Security Event ID 4688, and Sysmon Event ID 3 provided supporting evidence around the controlled execution.

The investigation also demonstrated an important DFIR principle: the existence of a suspicious-looking command or process does not automatically establish malicious intent. A reliable assessment requires correlation of execution context, user identity, timestamps, command-line information, recovery-system state, and subsequent activity.

## Skills Demonstrated

- Windows DFIR
- PowerShell Investigation
- Event Viewer
- VSS Investigation
- Windows Backup Investigation
- PowerShell Script Block Logging
- Windows Security Event 4688 Analysis
- Sysmon Event 3 Analysis
- Process Correlation
- Network Activity Analysis
- Timeline Construction
- Recovery Impact Assessment
- Evidence-Based Investigation
- MITRE ATT&CK Mapping

## Disclaimer

This investigation was performed in a controlled environment using simulated backup-destruction activity. No real organizational backup data or confidential information was used, and real recovery infrastructure was not intentionally destroyed.
