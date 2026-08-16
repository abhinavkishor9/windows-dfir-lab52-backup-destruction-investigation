# Timeline 

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

