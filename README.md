$Host.UI.RawUI.WindowTitle = "NONG KIM - Tweaking System..."

$InterfacesPath = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces"
Get-ChildItem -Path $InterfacesPath | ForEach-Object {
    $SubPath = $_.Name -replace "HKEY_LOCAL_MACHINE", "HKLM:"
    New-ItemProperty -Path $SubPath -Name "TcpAckFrequency" -Value 2 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
    New-ItemProperty -Path $SubPath -Name "TcpNoDelay" -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
    New-ItemProperty -Path $SubPath -Name "TcpWindowSize" -Value 65535 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
}

$TCPParameters = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters"
New-ItemProperty -Path $TCPParameters -Name "SackOpts" -Value 1 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path $TCPParameters -Name "TcpWindowSize" -Value 256960 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path $TCPParameters -Name "Tcp1323Opts" -Value 1 -PropertyType DWord -Force | Out-Null

$SystemProfile = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile"
New-ItemProperty -Path $SystemProfile -Name "NetworkThrottlingIndex" -Value 10 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path $SystemProfile -Name "SystemResponsiveness" -Value 20 -PropertyType DWord -Force | Out-Null

New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\USB" -Name "DisableUSB20HubPowerManagement" -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" -Name "ThumbnailLivePreviewHoverTime" -Value 0 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\GameDVR" -Name "AppCaptureEnabled" -Value 0 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKCU:\System\GameConfigStore" -Name "GameDVR_Enabled" -Value 0 -PropertyType DWord -Force | Out-Null

New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\PriorityControl" -Name "Win32PrioritySeparation" -Value 38 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" -Name "LargeSystemCache" -Value 0 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\dxgkrnl" -Name "MonitorLatencyTolerance" -Value 0 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" -Name "HwSchMode" -Value 2 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" -Name "TdrLevel" -Value 0 -PropertyType DWord -Force | Out-Null

$GameTaskPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks\Games"
New-ItemProperty -Path $GameTaskPath -Name "GPU Priority" -Value 8 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path $GameTaskPath -Name "Scheduling Category" -Value "High" -PropertyType String -Force | Out-Null

$KeyboardResponse = "HKCU:\Control Panel\Accessibility\KeyboardResponse"
if (!(Test-Path $KeyboardResponse)) { New-Item -Path $KeyboardResponse -Force | Out-Null }
New-ItemProperty -Path $KeyboardResponse -Name "Flags" -Value "126" -PropertyType String -Force | Out-Null
New-ItemProperty -Path $KeyboardResponse -Name "DelayBeforeAcceptance" -Value "0" -PropertyType String -Force | Out-Null
New-ItemProperty -Path $KeyboardResponse -Name "AutoRepeatDelay" -Value "150" -PropertyType String -Force | Out-Null
New-ItemProperty -Path $KeyboardResponse -Name "AutoRepeatRate" -Value "1" -PropertyType String -Force | Out-Null
New-ItemProperty -Path $KeyboardResponse -Name "BounceTime" -Value "0" -PropertyType String -Force | Out-Null

New-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name "KeyboardDelay" -Value "0" -PropertyType String -Force | Out-Null

Start-Sleep -Seconds 3

Clear-Host
$Host.UI.RawUI.WindowTitle = "NONG KIM - Optimization Completed!"

Write-Host @"
=========================================================
  _  _  ___  _  _  ___     _  __ ___ __  __

 | \| |/ _ \| \| |/ __|   | |/ /|_ _|  \/  |
 | .  | (_) | .  | (_ |   |   <  | || |\/| |
 |_|\_|\___/|_|\_|\___|   |_|\_\|___|_|  |_|
=========================================================
"@ -ForegroundColor Cyan

Write-Host " [✓] ALL REGISTRY TWEAKS APPLIED SUCCESSFULLY! (ABYSS)" -ForegroundColor Green
Write-Host " -------------------------------------------------------" -ForegroundColor Cyan

for ($i = 5; $i -gt 0; $i--) {
    Write-Host "`r [!] Your PC will restart automatically in $i seconds... " -ForegroundColor Yellow -NoNewline
    Start-Sleep -Seconds 1
}
Write-Host "`r [!] Restarting Windows Now!                               " -ForegroundColor Red

Restart-Computer -Force
