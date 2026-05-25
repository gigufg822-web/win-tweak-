$Host.UI.RawUI.WindowTitle = "NONG KIM - System Verification"
Clear-Host

$CorrectKey = "nongkim"

function Get-MaskedInput {
    param ([string]$Prompt = "Enter Access Key: ")
    Write-Host $Prompt -NoNewline -ForegroundColor Cyan
    $Password = New-Object System.Security.SecureString
    while ($true) {
        $Key = [Console]::ReadKey($true)
        if ($Key.Key -eq [ConsoleKey]::Enter) {
            Write-Host ""
            break
        }
        elseif ($Key.Key -eq [ConsoleKey]::Backspace) {
            if ($Password.Length -gt 0) {
                $Password.RemoveAt($Password.Length - 1)
                Write-Host "`b `b" -NoNewline
            }
        }
        elseif ($Key.KeyChar -ne `0) {
            $Password.AppendChar($Key.KeyChar)
            Write-Host "*" -NoNewline -ForegroundColor Yellow
        }
    }
    $BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($Password)
    $PlainPassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)
    return $PlainPassword
}

$UserKey = Get-MaskedInput -Prompt "Enter Key: "

if ($UserKey -ne $CorrectKey) {
    Write-Host "`n [X] Incorrect Key! Exiting..." -ForegroundColor Red
    Start-Sleep -Seconds 3
    Exit
}

$Host.UI.RawUI.WindowTitle = "NONG KIM - Tweaking System..."
Write-Host "`n [✓] Access Granted! Starting optimization..." -ForegroundColor Green
Start-Sleep -Seconds 1

$InterfacesPath = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces"
Get-ChildItem -Path $InterfacesPath | ForEach-Object {
    $SubPath = $_.Name -replace "HKEY_LOCAL_MACHINE", "HKLM:"
    New-ItemProperty -Path $SubPath -Name "TcpAckFrequency" -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
    New-ItemProperty -Path $SubPath -Name "TcpNoDelay" -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
    New-ItemProperty -Path $SubPath -Name "TcpWindowSize" -Value 65535 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
}

$TCPParameters = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters"
New-ItemProperty -Path $TCPParameters -Name "SackOpts" -Value 1 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path $TCPParameters -Name "TcpWindowSize" -Value 256960 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path $TCPParameters -Name "Tcp1323Opts" -Value 1 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path $TCPParameters -Name "DefaultTTL" -Value 64 -PropertyType DWord -Force | Out-Null

$SystemProfile = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile"
New-ItemProperty -Path $SystemProfile -Name "NetworkThrottlingIndex" -Value 4294967295 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path $SystemProfile -Name "SystemResponsiveness" -Value 0 -PropertyType DWord -Force | Out-Null

New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\USB" -Name "DisableUSB20HubPowerManagement" -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\BTHPORT\Parameters" -Name "BluetoothControl" -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null

New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced" -Name "ThumbnailLivePreviewHoverTime" -Value 0 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\GameDVR" -Name "AppCaptureEnabled" -Value 0 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKCU:\System\GameConfigStore" -Name "GameDVR_Enabled" -Value 0 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKCU:\System\GameConfigStore" -Name "GameDVR_FSEBehaviorMode" -Value 2 -PropertyType DWord -Force | Out-Null

New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\PriorityControl" -Name "Win32PrioritySeparation" -Value 40 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" -Name "LargeSystemCache" -Value 0 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\dxgkrnl" -Name "MonitorLatencyTolerance" -Value 0 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" -Name "HwSchMode" -Value 2 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" -Name "TdrLevel" -Value 0 -PropertyType DWord -Force | Out-Null

$GameTaskPath = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks\Games"
New-ItemProperty -Path $GameTaskPath -Name "GPU Priority" -Value 18 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path $GameTaskPath -Name "Priority" -Value 6 -PropertyType DWord -Force | Out-Null
New-ItemProperty -Path $GameTaskPath -Name "Scheduling Category" -Value "High" -PropertyType String -Force | Out-Null

$KeyboardResponse = "HKCU:\Control Panel\Accessibility\KeyboardResponse"
if (!(Test-Path $KeyboardResponse)) { New-Item -Path $KeyboardResponse -Force | Out-Null }
New-ItemProperty -Path $KeyboardResponse -Name "Flags" -Value "126" -PropertyType String -Force | Out-Null
New-ItemProperty -Path $KeyboardResponse -Name "DelayBeforeAcceptance" -Value "0" -PropertyType String -Force | Out-Null
New-ItemProperty -Path $KeyboardResponse -Name "AutoRepeatDelay" -Value "140" -PropertyType String -Force | Out-Null
New-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name "KeyboardDelay" -Value "0" -PropertyType String -Force | Out-Null
New-ItemProperty -Path "HKCU:\Control Panel\Desktop" -Name "KeyboardSpeed" -Value "31" -PropertyType String -Force | Out-Null

$MousePath = "HKCU:\Control Panel\Mouse"
New-ItemProperty -Path $MousePath -Name "MouseSpeed" -Value "0" -PropertyType String -Force | Out-Null
New-ItemProperty -Path $MousePath -Name "MouseThreshold1" -Value "0" -PropertyType String -Force | Out-Null
New-ItemProperty -Path $MousePath -Name "MouseThreshold2" -Value "0" -PropertyType String -Force | Out-Null

$UnnecessaryServices = @(
    "DiagTrack",
    "dmwappushservice",
    "WerSvc",
    "PcaSvc",
    "lfsvc",
    "TrkWks",
    "WbioSrvc",
    "ParentalControls",
    "XblAuthManager",
    "XblGameSave",
    "XboxNetApiSvc",
    "MapsBroker"
)

foreach ($Service in $UnnecessaryServices) {
    if (Get-Service -Name $Service -ErrorAction SilentlyContinue) {
        Stop-Service -Name $Service -Force -ErrorAction SilentlyContinue
        Set-Service -Name $Service -StartupType Disabled -ErrorAction SilentlyContinue
    }
}

Start-Sleep -Seconds 3
Clear-Host
$Host.UI.RawUI.WindowTitle = "NONG KIM - Optimization Completed!"

Write-Host @"
=========================================================
  _  _  ___  _  _  ___     _  __ ___ __  __  

 | \| |/ _ \| \| |/ __|   | |/ /|_ _|  \/  | 
 | .  | (_) | .  | (_ |   |  <  | || |\/| | 
 |_|\_|\___/|_|\_|\___|   |_|\_\|___|_|  |_| 
=========================================================
"@ -ForegroundColor Cyan

Write-Host " [✓] ALL REGISTRY TWEAKS APPLIED SUCCESSFULLY! (nongkim)" -ForegroundColor Green
Write-Host " -------------------------------------------------------" -ForegroundColor Cyan

for ($i = 5; $i -gt 0; $i--) {
    Write-Host "`r [!] Your PC will restart automatically in $i seconds... " -ForegroundColor Yellow -NoNewline
    Start-Sleep -Seconds 1
}
Write-Host "`r [!] Restarting Windows Now!                               " -ForegroundColor Red

Restart-Computer -Force
