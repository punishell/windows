# windows


```
# Safe Mode Admin PowerShell Script
$ErrorActionPreference = 'SilentlyContinue'

# Bypass Tamper Protection via registry
$regPath = "HKLM:\SOFTWARE\Microsoft\Windows Defender\Features"
$regValue = "TamperProtection"
if (!(Test-Path $regPath)) { New-Item -Path $regPath -Force }
Set-ItemProperty -Path $regPath -Name $regValue -Value 0 -Type DWORD -Force

# Take full ownership of Defender registry keys
$keys = @(
    "HKLM:\SOFTWARE\Microsoft\Windows Defender",
    "HKLM:\SYSTEM\CurrentControlSet\Services\WinDefend",
    "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender"
)

foreach ($key in $keys) {
    takeown /f $key.Replace('HKLM:\', 'HKLM\') /A
    icacls $key.Replace('HKLM:\', 'HKLM\') /grant "Administrators":F /T /C
}

# Force kill Defender processes
Stop-Process -Name "MsMpEng" -Force
Stop-Process -Name "NisSrv" -Force

# Disable services via registry (bypassing service manager)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\WinDefend" -Name "Start" -Value 4 -Type DWORD -Force
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\wscsvc" -Name "Start" -Value 4 -Type DWORD -Force
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\SecurityHealthService" -Name "Start" -Value 4 -Type DWORD -Force

# Disable through Group Policy
$gpoPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender"
if (!(Test-Path $gpoPath)) { New-Item -Path $gpoPath -Force }
Set-ItemProperty -Path $gpoPath -Name "DisableAntiSpyware" -Value 1 -Type DWORD -Force

# Disable all Defender scheduled tasks
Get-ScheduledTask -TaskPath "\Microsoft\Windows\Windows Defender\" | Disable-ScheduledTask

# Final cleanup
Remove-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows Defender\Scan" -Recurse -Force
Remove-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows Defender\Features" -Recurse -Force

Write-Host "Windows Defender has been completely disabled. Please restart your computer."

# WARNING: This is irreversible!
Disable-WindowsOptionalFeature -Online -FeatureName "Windows-Defender" -NoRestart
Remove-WindowsCapability -Online -Name "Windows.Defender~~~~0.0.1.0"
```
