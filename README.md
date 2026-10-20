Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing
Add-Type -TypeDefinition @"
using System;
using System.Runtime.InteropServices;
public class DPI {
    [DllImport("user32.dll")]
    public static extern bool SetProcessDPIAware();
}
public class NativeMethods {
    [DllImport("ntdll.dll")]
    public static extern int NtSetTimerResolution(uint DesiredResolution, bool SetResolution, out uint CurrentResolution);
}
"@
[DPI]::SetProcessDPIAware()

$form = New-Object System.Windows.Forms.Form
$form.Text = "67 - Ultimate CPU / GPU & FPS Boost"
$form.Size = New-Object System.Drawing.Size(520, 540)
$form.StartPosition = 'CenterScreen'
$form.BackColor = [System.Drawing.Color]::Black
$form.FormBorderStyle = 'FixedDialog'
$form.MaximizeBox = $false

$fontTitle = New-Object System.Drawing.Font("Impact", 34, [System.Drawing.FontStyle]::Regular)
$fontNormal = New-Object System.Drawing.Font("Consolas", 11, [System.Drawing.FontStyle]::Regular)

$lblTitle = New-Object System.Windows.Forms.Label
$lblTitle.Text = "NONGKIM"
$lblTitle.Font = $fontTitle
$lblTitle.ForeColor = [System.Drawing.Color]::White
$lblTitle.Size = New-Object System.Drawing.Size(420, 55)
$lblTitle.Location = New-Object System.Drawing.Point(50, 25)
$lblTitle.TextAlign = 'MiddleCenter'
$form.Controls.Add($lblTitle)

$lblSub = New-Object System.Windows.Forms.Label
$lblSub.Text = "ENTER KEY TO UNLOCK"
$lblSub.Font = $fontNormal
$lblSub.ForeColor = [System.Drawing.Color]::DimGray
$lblSub.Size = New-Object System.Drawing.Size(420, 25)
$lblSub.Location = New-Object System.Drawing.Point(50, 85)
$lblSub.TextAlign = 'MiddleCenter'
$form.Controls.Add($lblSub)

# --- ส่วนใส่ Key ปลดล็อก ---
$txtKey = New-Object System.Windows.Forms.TextBox
$txtKey.Location = New-Object System.Drawing.Point(160, 130)
$txtKey.Size = New-Object System.Drawing.Size(200, 30)
$txtKey.PasswordChar = '*'
$txtKey.BackColor = [System.Drawing.Color]::FromArgb(20, 20, 20)
$txtKey.ForeColor = [System.Drawing.Color]::White
$txtKey.TextAlign = 'Center'
$form.Controls.Add($txtKey)

$btnUnlock = New-Object System.Windows.Forms.Button
$btnUnlock.Text = "UNLOCK KEY"
$btnUnlock.Location = New-Object System.Drawing.Point(160, 175)
$btnUnlock.Size = New-Object System.Drawing.Size(200, 35)
$btnUnlock.BackColor = [System.Drawing.Color]::FromArgb(40, 40, 40)
$btnUnlock.ForeColor = [System.Drawing.Color]::White
$btnUnlock.FlatStyle = 'Flat'
$form.Controls.Add($btnUnlock)

# --- ปุ่มเมนูควบคุม (ซ่อนไว้ก่อนจนกว่าจะใส่ Key ถูกต้อง) ---
$rbApply = New-Object System.Windows.Forms.RadioButton
$rbApply.Text = "APPLY ALL"
$rbApply.Location = New-Object System.Drawing.Point(160, 225)
$rbApply.Size = New-Object System.Drawing.Size(100, 30)
$rbApply.ForeColor = [System.Drawing.Color]::White
$rbApply.Checked = $true
$rbApply.Visible = $false
$form.Controls.Add($rbApply)

$rbReset = New-Object System.Windows.Forms.RadioButton
$rbReset.Text = "RESET"
$rbReset.Location = New-Object System.Drawing.Point(270, 225)
$rbReset.Size = New-Object System.Drawing.Size(80, 30)
$rbReset.ForeColor = [System.Drawing.Color]::White
$rbReset.Visible = $false
$form.Controls.Add($rbReset)

$btnRun = New-Object System.Windows.Forms.Button
$btnRun.Text = "EXECUTE FULL BOOST"
$btnRun.Location = New-Object System.Drawing.Point(160, 265)
$btnRun.Size = New-Object System.Drawing.Size(200, 40)
$btnRun.BackColor = [System.Drawing.Color]::DarkGreen
$btnRun.ForeColor = [System.Drawing.Color]::White
$btnRun.FlatStyle = 'Flat'
$btnRun.Visible = $false
$form.Controls.Add($btnRun)

$lblStatus = New-Object System.Windows.Forms.Label
$lblStatus.Size = New-Object System.Drawing.Size(460, 40)
$lblStatus.Location = New-Object System.Drawing.Point(30, 325)
$lblStatus.TextAlign = 'MiddleCenter'
$lblStatus.ForeColor = [System.Drawing.Color]::Gray
$form.Controls.Add($lblStatus)

# --- Event ปลดล็อก Key ---
$btnUnlock.Add_Click({
    if ($txtKey.Text -eq "67") {
        $lblStatus.Text = "KEY ACCEPTED: UNLOCKED"; $lblStatus.ForeColor = [System.Drawing.Color]::Green
        $lblSub.Text = "MAXIMUM CPU / GPU / FPS BOOST"
        $txtKey.Enabled = $false
        $btnUnlock.Enabled = $false
        $btnUnlock.Visible = $false
        
        $rbApply.Visible = $true
        $rbReset.Visible = $true
        $btnRun.Visible = $true
    } else {
        $lblStatus.Text = "ACCESS DENIED: INVALID KEY"; $lblStatus.ForeColor = [System.Drawing.Color]::DarkRed
        $txtKey.Clear()
    }
})

# --- Event รันสคริปต์หลัก ---
$btnRun.Add_Click({
    $btnRun.Enabled = $false; $rbApply.Enabled = $false; $rbReset.Enabled = $false
    $OldErrorActionPreference = $ErrorActionPreference
    $ErrorActionPreference = "Stop"

    try {
        $lblStatus.Text = "APPLYING FULL CPU/GPU/SYS BOOST..."; $lblStatus.ForeColor = [System.Drawing.Color]::Yellow; 
        [System.Windows.Forms.Application]::DoEvents()

        if ($rbApply.Checked) {
            # =========================================================
            # --- 1. CPU OPTIMIZATIONS (Core Parking & Power) ---
            # =========================================================
            # Disable Core Parking (Keep all CPU cores active)
            $PowerControl = "HKLM:\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\54533751-8270-4e01-9723-171092d09440\0cc5b647-c1df-4424-92f1-47f5ca73409c"
            New-ItemProperty -Path $PowerControl -Name "Attributes" -Value 0 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
            powercfg -setacvalueindex SCHEME_CURRENT SUB_PROCESSOR 0cc5b647-c1df-4424-92f1-47f5ca73409c 100 2>$null
            
            # Force Ultimate Performance Power Plan
            $ultimatePlan = powercfg -duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61 2>&1
            if ($ultimatePlan -match '([a-f0-9\-]{36})') {
                powercfg -setactive $Matches[1] 2>$null
            }

            # Win32PrioritySeparation = 38 (0x26 Hex: Highest priority to Foreground Games)
            New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\PriorityControl" -Name "Win32PrioritySeparation" -Value 38 -PropertyType DWord -Force | Out-Null

            # =========================================================
            # --- 2. GPU OPTIMIZATIONS (Hardware Scheduling & Drivers) ---
            # =========================================================
            # Enable Hardware-Accelerated GPU Scheduling (HAGS = 2)
            New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" -Name "HwSchMode" -Value 2 -PropertyType DWord -Force | Out-Null
            
            # Disable GPU Energy Saving & Set Max Performance Profile
            New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" -Name "DisableBlocklist" -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
            
            # Multimedia Profile Games Task Priority (GPU = 8, CPU Priority = 6)
            $GamesTask = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks\Games"
            New-ItemProperty $GamesTask -Name "GPU Priority" -Value 8 -PropertyType DWord -Force | Out-Null
            New-ItemProperty $GamesTask -Name "Priority" -Value 6 -PropertyType DWord -Force | Out-Null
            New-ItemProperty $GamesTask -Name "Scheduling Category" -Value "High" -PropertyType String -Force | Out-Null
            New-ItemProperty $GamesTask -Name "SFIO Priority" -Value "High" -PropertyType String -Force | Out-Null

            # =========================================================
            # --- 3. WINDOWS GAME MODE & BLOATWARE TWEAKS ---
            # =========================================================
            Set-ItemProperty "HKCU:\Software\Microsoft\GameBar" -Name "AutoGameModeEnabled" -Value 1 -Force -ErrorAction SilentlyContinue
            New-Item -Path "HKCU:\System\GameConfigStore" -Force | Out-Null
            Set-ItemProperty "HKCU:\System\GameConfigStore" -Name "GameDVR_Enabled" -Value 0 -Force -ErrorAction SilentlyContinue
            Set-ItemProperty "HKCU:\System\GameConfigStore" -Name "GameDVR_FSEBehaviorMode" -Value 2 -Force -ErrorAction SilentlyContinue
            Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\GameDVR" -Name "AppCaptureEnabled" -Value 0 -Force -ErrorAction SilentlyContinue

            # Performance Visual Effects (Best Performance)
            Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects" -Name "VisualFXSetting" -Value 2 -Force -ErrorAction SilentlyContinue
            
            # Disable Background Apps globally
            New-Item -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications" -Force | Out-Null
            Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications" -Name "GlobalUserDisabled" -Value 1 -Force -ErrorAction SilentlyContinue

            # =========================================================
            # --- 4. NETWORK & TCP/IP LOW LATENCY ---
            # =========================================================
            $TCP = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters"
            $NetSettings = @{
                "TcpWindowSize" = 65535; "GlobalMaxTcpWindowSize" = 65535; "Tcp1323Opts" = 1; "DefaultTTL" = 64; 
                "SackOpts" = 1; "TcpMaxDataRetransmissions" = 2; "SynAttackProtect" = 0; 
                "TcpNumConnections" = 16777214; "TcpTimedWaitDelay" = 30; "EnableCompoundTcp" = 1;
                "MaxUserPort" = 65534; "MaxFreeTcbs" = 65536; "MaxHashTableSize" = 65536; "DisableTaskOffload" = 0
            }
            foreach ($Key in $NetSettings.Keys) { New-ItemProperty $TCP -Name $Key -Value $NetSettings[$Key] -PropertyType DWord -Force | Out-Null }
            netsh int tcp set global autotuninglevel=normal rss=enabled chimney=enabled netdma=enabled congestionprovider=ctcp ecncapability=enabled timestamps=disabled | Out-Null
            
            $Adapter = Get-NetAdapter | Where-Object {$_.Status -eq "Up"} | Select-Object -First 1
            if ($Adapter) { 
                Set-NetAdapterAdvancedProperty -Name $Adapter.Name -DisplayName "Interrupt Moderation" -DisplayValue "Disabled" -ErrorAction SilentlyContinue
                Set-NetAdapterAdvancedProperty -Name $Adapter.Name -DisplayName "Flow Control" -DisplayValue "Disabled" -ErrorAction SilentlyContinue 
            }
            
            New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\AFD\Parameters" -Name "DefaultReceiveWindow" -Value 65535 -PropertyType DWord -Force | Out-Null
            New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\AFD\Parameters" -Name "DefaultSendWindow" -Value 65535 -PropertyType DWord -Force | Out-Null
            New-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" -Name "NetworkThrottlingIndex" -Value 4294967295 -PropertyType DWord -Force | Out-Null
            New-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" -Name "SystemResponsiveness" -Value 0 -PropertyType DWord -Force | Out-Null

            # =========================================================
            # --- 5. MOUSE & PERSISTENT FILTERKEYS (Flags 1, Delay 150, Rate 1) ---
            # =========================================================
            Set-ItemProperty "HKCU:\Control Panel\Mouse" -Name "MouseSpeed" -Value 0
            Set-ItemProperty "HKCU:\Control Panel\Mouse" -Name "MouseThreshold1" -Value 0
            Set-ItemProperty "HKCU:\Control Panel\Mouse" -Name "MouseThreshold2" -Value 0
            
            $fkPath = "HKCU:\Control Panel\Accessibility\Keyboard Response"
            Set-ItemProperty $fkPath -Name "Flags" -Value "1"
            Set-ItemProperty $fkPath -Name "AutoRepeatDelay" -Value "150"
            Set-ItemProperty $fkPath -Name "AutoRepeatRate" -Value "1"
            Set-ItemProperty $fkPath -Name "BounceTime" -Value "0"
            Set-ItemProperty $fkPath -Name "DelayBeforeAcceptance" -Value "0"

            $defFkPath = "Registry::HKEY_USERS\.DEFAULT\Control Panel\Accessibility\Keyboard Response"
            if (Test-Path "Registry::HKEY_USERS\.DEFAULT") {
                Set-ItemProperty $defFkPath -Name "Flags" -Value "1" -ErrorAction SilentlyContinue
                Set-ItemProperty $defFkPath -Name "AutoRepeatDelay" -Value "150" -ErrorAction SilentlyContinue
                Set-ItemProperty $defFkPath -Name "AutoRepeatRate" -Value "1" -ErrorAction SilentlyContinue
                Set-ItemProperty $defFkPath -Name "BounceTime" -Value "0" -ErrorAction SilentlyContinue
                Set-ItemProperty $defFkPath -Name "DelayBeforeAcceptance" -Value "0" -ErrorAction SilentlyContinue
            }

            # =========================================================
            # --- 6. CLEANUP & SERVICES ---
            # =========================================================
            foreach ($Svc in @("DiagTrack", "dmwappushservice", "SysMain", "MapsBroker", "Fax")) { 
                Set-Service $Svc -StartupType Disabled -ErrorAction SilentlyContinue
                Stop-Service $Svc -Force -ErrorAction SilentlyContinue 
            }
            Remove-Item -Path "$env:TEMP\*", "C:\Windows\Temp\*" -Recurse -Force -ErrorAction SilentlyContinue
            [System.GC]::Collect()

            # =========================================================
            # --- 7. KERNEL & PERSISTENT 0.5MS TIMER RESOLUTION ---
            # =========================================================
            New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Kernel" -Name "GlobalTimerResolutionRequests" -Value 1 -PropertyType DWord -Force | Out-Null
            bcdedit /set disabledynamictick yes | Out-Null
            bcdedit /deletevalue useplatformclock 2>$null | Out-Null
            
            [uint32]$currentRes = 0
            [NativeMethods]::NtSetTimerResolution(5000, $true, [ref]$currentRes) | Out-Null

            $Action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-Command `"[uint32]`$r=0; [NativeMethods]::NtSetTimerResolution(5000, `$true, [ref]`$r)`""
            $Trigger = New-ScheduledTaskTrigger -AtLogOn
            $Principal = New-ScheduledTaskPrincipal -UserId "$env:USERNAME" -RunLevel Highest
            Register-ScheduledTask -TaskName "NONGKIM_TimerResolution" -Action $Action -Trigger $Trigger -Principal$Principal -Force | Out-Null

        } else {
            # --- RESET MODE ---
            $TCP = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters"
            Remove-ItemProperty $TCP -Name "TcpWindowSize", "GlobalMaxTcpWindowSize", "Tcp1323Opts", "DefaultTTL", "SackOpts", "TcpMaxDataRetransmissions", "SynAttackProtect", "TcpNumConnections", "TcpTimedWaitDelay", "EnableCompoundTcp", "MaxUserPort", "MaxFreeTcbs", "MaxHashTableSize", "DisableTaskOffload" -ErrorAction SilentlyContinue
            netsh int tcp set global autotuninglevel=disabled | Out-Null
            
            Set-ItemProperty "HKCU:\Control Panel\Accessibility\Keyboard Response" -Name "Flags" -Value "126"

            Remove-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Kernel" -Name "GlobalTimerResolutionRequests" -ErrorAction SilentlyContinue
            bcdedit /deletevalue disabledynamictick 2>$null | Out-Null
            
            [uint32]$currentRes = 0
            [NativeMethods]::NtSetTimerResolution(5000, $false, [ref]$currentRes) | Out-Null

            Unregister-ScheduledTask -TaskName "NONGKIM_TimerResolution" -Confirm:$false -ErrorAction SilentlyContinue
        }
        
        $lblStatus.Text = "SUCCESS: FULL CPU/GPU BOOST APPLIED"; $lblStatus.ForeColor = [System.Drawing.Color]::Green
    }
    catch {
        $ShortError =$_.Exception.Message
        if ($ShortError.Length -gt 35) { $ShortError =$ShortError.Substring(0,35) + "..." }
        $lblStatus.Text = "FAILED: $ShortError"; $lblStatus.ForeColor = [System.Drawing.Color]::DarkRed
    }
    finally {
        $btnRun.Enabled =$true; $rbApply.Enabled =$true; $rbReset.Enabled =$true
        $ErrorActionPreference =$OldErrorActionPreference
    }
})

[void]$form.ShowDialog()
