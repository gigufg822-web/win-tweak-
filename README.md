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

# --- ตั้งค่าหน้าต่าง UI หลัก ---
$form = New-Object System.Windows.Forms.Form
$form.Text = "NONGKIM"
$form.Size = New-Object System.Drawing.Size(560, 440)
$form.StartPosition = 'CenterScreen'
$form.BackColor = [System.Drawing.Color]::FromArgb(15, 15, 15)
$form.FormBorderStyle = 'FixedDialog'
$form.MaximizeBox = $false

# --- แบบฟอนต์ ---
$fontTitle  = New-Object System.Drawing.Font("Impact", 44, [System.Drawing.FontStyle]::Regular)
$fontButton = New-Object System.Drawing.Font("Consolas", 11, [System.Drawing.FontStyle]::Bold)
$fontRadio  = New-Object System.Drawing.Font("Consolas", 11, [System.Drawing.FontStyle]::Regular)
$fontStatus = New-Object System.Drawing.Font("Consolas", 11, [System.Drawing.FontStyle]::Bold)

# --- ข้อความหลัก NONGKIM ---
$lblTitle = New-Object System.Windows.Forms.Label
$lblTitle.Text = "NONGKIM"
$lblTitle.Font = $fontTitle
$lblTitle.ForeColor = [System.Drawing.Color]::White
$lblTitle.Size = New-Object System.Drawing.Size(500, 70)
$lblTitle.Location = New-Object System.Drawing.Point(30, 30)
$lblTitle.TextAlign = 'MiddleCenter'
$form.Controls.Add($lblTitle)

# --- ช่องใส่ Key ปลดล็อก ---
$txtKey = New-Object System.Windows.Forms.TextBox
$txtKey.Location = New-Object System.Drawing.Point(170, 130)
$txtKey.Size = New-Object System.Drawing.Size(220, 30)
$txtKey.PasswordChar = '*'
$txtKey.Font = $fontButton
$txtKey.BackColor = [System.Drawing.Color]::FromArgb(30, 30, 30)
$txtKey.ForeColor = [System.Drawing.Color]::White
$txtKey.TextAlign = 'Center'
$form.Controls.Add($txtKey)

$btnUnlock = New-Object System.Windows.Forms.Button
$btnUnlock.Text = "UNLOCK KEY"
$btnUnlock.Font = $fontButton
$btnUnlock.Location = New-Object System.Drawing.Point(170, 175)
$btnUnlock.Size = New-Object System.Drawing.Size(220, 40)
$btnUnlock.BackColor = [System.Drawing.Color]::FromArgb(45, 45, 45)
$btnUnlock.ForeColor = [System.Drawing.Color]::White
$btnUnlock.FlatStyle = 'Flat'
$btnUnlock.FlatAppearance.BorderSize = 0
$form.Controls.Add($btnUnlock)

# --- Radio Buttons (ซ่อนไว้ก่อนปลดล็อก) ---
$rbApply = New-Object System.Windows.Forms.RadioButton
$rbApply.Text = "APPLY ALL"
$rbApply.Font = $fontRadio
$rbApply.Location = New-Object System.Drawing.Point(165, 130)
$rbApply.Size = New-Object System.Drawing.Size(130, 30)
$rbApply.ForeColor = [System.Drawing.Color]::White
$rbApply.Checked = $true
$rbApply.Visible = $false
$form.Controls.Add($rbApply)

$rbReset = New-Object System.Windows.Forms.RadioButton
$rbReset.Text = "RESET"
$rbReset.Font = $fontRadio
$rbReset.Location = New-Object System.Drawing.Point(315, 130)
$rbReset.Size = New-Object System.Drawing.Size(90, 30)
$rbReset.ForeColor = [System.Drawing.Color]::White
$rbReset.Visible = $false
$form.Controls.Add($rbReset)

# --- ปุ่มสั่งรันหลัก (ซ่อนไว้ก่อนปลดล็อก) ---
$btnRun = New-Object System.Windows.Forms.Button
$btnRun.Text = "EXECUTE FULL BOOST"
$btnRun.Font = $fontButton
$btnRun.Location = New-Object System.Drawing.Point(150, 180)
$btnRun.Size = New-Object System.Drawing.Size(260, 45)
$btnRun.BackColor = [System.Drawing.Color]::FromArgb(0, 128, 0)
$btnRun.ForeColor = [System.Drawing.Color]::White
$btnRun.FlatStyle = 'Flat'
$btnRun.FlatAppearance.BorderSize = 0
$btnRun.Visible = $false
$form.Controls.Add($btnRun)

# --- ป้ายแสดงสถานะ ---
$lblStatus = New-Object System.Windows.Forms.Label
$lblStatus.Font = $fontStatus
$lblStatus.Size = New-Object System.Drawing.Size(500, 40)
$lblStatus.Location = New-Object System.Drawing.Point(30, 275)
$lblStatus.TextAlign = 'MiddleCenter'
$lblStatus.ForeColor = [System.Drawing.Color]::Gray
$form.Controls.Add($lblStatus)

# --- Event ปลดล็อก Key ---
$btnUnlock.Add_Click({
    if ($txtKey.Text -eq "67") {
        $lblStatus.Text = "UNLOCKED"; $lblStatus.ForeColor = [System.Drawing.Color]::LimeGreen
        $txtKey.Visible = $false
        $btnUnlock.Visible = $false
        
        $rbApply.Visible = $true
        $rbReset.Visible = $true
        $btnRun.Visible = $true
    } else {
        $lblStatus.Text = "INVALID KEY"; $lblStatus.ForeColor = [System.Drawing.Color]::Crimson
        $txtKey.Clear()
    }
})

# --- Event รันสคริปต์หลัก ---
$btnRun.Add_Click({
    $btnRun.Enabled = $false; $rbApply.Enabled = $false; $rbReset.Enabled = $false
    $OldErrorActionPreference = $ErrorActionPreference
    $ErrorActionPreference = "Stop"

    try {
        $lblStatus.Text = "APPLYING BOOST..."; $lblStatus.ForeColor = [System.Drawing.Color]::Khaki; 
        [System.Windows.Forms.Application]::DoEvents()

        if ($rbApply.Checked) {
            # 1. CPU Power Settings
            try {
                $PowerControl = "HKLM:\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\54533751-8270-4e01-9723-171092d09440\0cc5b647-c1df-4424-92f1-47f5ca73409c"
                if (Test-Path $PowerControl) {
                    New-ItemProperty -Path $PowerControl -Name "Attributes" -Value 0 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
                    powercfg -setacvalueindex SCHEME_CURRENT SUB_PROCESSOR 0cc5b647-c1df-4424-92f1-47f5ca73409c 100 2>&1 | Out-Null
                }
            } catch {}
            
            try {
                $ultimatePlan = powercfg -duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61 2>&1
                if ($ultimatePlan -match '([a-f0-9\-]{36})') {
                    powercfg -setactive $Matches[1] 2>&1 | Out-Null
                }
            } catch {}

            New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\PriorityControl" -Name "Win32PrioritySeparation" -Value 38 -PropertyType DWord -Force | Out-Null

            # 2. GPU Settings
            New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" -Name "HwSchMode" -Value 2 -PropertyType DWord -Force | Out-Null
            New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" -Name "DisableBlocklist" -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
            
            $GamesTask = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks\Games"
            New-ItemProperty $GamesTask -Name "GPU Priority" -Value 8 -PropertyType DWord -Force | Out-Null
            New-ItemProperty $GamesTask -Name "Priority" -Value 6 -PropertyType DWord -Force | Out-Null
            New-ItemProperty $GamesTask -Name "Scheduling Category" -Value "High" -PropertyType String -Force | Out-Null
            New-ItemProperty $GamesTask -Name "SFIO Priority" -Value "High" -PropertyType String -Force | Out-Null

            # 3. Game Mode & Visuals
            Set-ItemProperty "HKCU:\Software\Microsoft\GameBar" -Name "AutoGameModeEnabled" -Value 1 -Force -ErrorAction SilentlyContinue
            New-Item -Path "HKCU:\System\GameConfigStore" -Force -ErrorAction SilentlyContinue | Out-Null
            Set-ItemProperty "HKCU:\System\GameConfigStore" -Name "GameDVR_Enabled" -Value 0 -Force -ErrorAction SilentlyContinue
            Set-ItemProperty "HKCU:\System\GameConfigStore" -Name "GameDVR_FSEBehaviorMode" -Value 2 -Force -ErrorAction SilentlyContinue
            Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\GameDVR" -Name "AppCaptureEnabled" -Value 0 -Force -ErrorAction SilentlyContinue
            Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects" -Name "VisualFXSetting" -Value 2 -Force -ErrorAction SilentlyContinue
            
            New-Item -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications" -Force -ErrorAction SilentlyContinue | Out-Null
            Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications" -Name "GlobalUserDisabled" -Value 1 -Force -ErrorAction SilentlyContinue

            # 4. Network Optimization
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

            # 5. Input Lag & FilterKeys Tweaks (กดเร็ว ตอบสนองไว)
            Set-ItemProperty "HKCU:\Control Panel\Mouse" -Name "MouseSpeed" -Value 0 -ErrorAction SilentlyContinue
            Set-ItemProperty "HKCU:\Control Panel\Mouse" -Name "MouseThreshold1" -Value 0 -ErrorAction SilentlyContinue
            Set-ItemProperty "HKCU:\Control Panel\Mouse" -Name "MouseThreshold2" -Value 0 -ErrorAction SilentlyContinue
            
            $fkPath = "HKCU:\Control Panel\Accessibility\Keyboard Response"
            Set-ItemProperty $fkPath -Name "Flags" -Value "1" -ErrorAction SilentlyContinue
            Set-ItemProperty $fkPath -Name "AutoRepeatDelay" -Value "150" -ErrorAction SilentlyContinue
            Set-ItemProperty $fkPath -Name "AutoRepeatRate" -Value "1" -ErrorAction SilentlyContinue
            Set-ItemProperty $fkPath -Name "BounceTime" -Value "0" -ErrorAction SilentlyContinue
            Set-ItemProperty $fkPath -Name "DelayBeforeAcceptance" -Value "0" -ErrorAction SilentlyContinue

            # 6. Cleanup & Services
            foreach ($Svc in @("DiagTrack", "dmwappushservice", "SysMain", "MapsBroker", "Fax")) { 
                Set-Service $Svc -StartupType Disabled -ErrorAction SilentlyContinue
                Stop-Service $Svc -Force -ErrorAction SilentlyContinue 
            }
            Remove-Item -Path "$env:TEMP\*", "C:\Windows\Temp\*" -Recurse -Force -ErrorAction SilentlyContinue
            [System.GC]::Collect()

            # 7. Timer Resolution
            New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Kernel" -Name "GlobalTimerResolutionRequests" -Value 1 -PropertyType DWord -Force | Out-Null
            bcdedit /set disabledynamictick yes 2>&1 | Out-Null
            bcdedit /deletevalue useplatformclock 2>&1 | Out-Null
            
            [uint32]$currentRes = 0
            [NativeMethods]::NtSetTimerResolution(5000, $true, [ref]$currentRes) | Out-Null

            $schtasksArg = '/create /tn "NONGKIM_TimerResolution" /tr "powershell.exe -WindowStyle Hidden -Command \"[uint32]`$r=0; [NativeMethods]::NtSetTimerResolution(5000, `$true, [ref]`$r)\"" /sc ONLOGON /rl HIGHEST /f'
            Start-Process schtasks.exe -ArgumentList $schtasksArg -Wait -NoNewWindow -ErrorAction SilentlyContinue

        } else {
            # --- RESET MODE ---
            $TCP = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters"
            Remove-ItemProperty $TCP -Name "TcpWindowSize", "GlobalMaxTcpWindowSize", "Tcp1323Opts", "DefaultTTL", "SackOpts", "TcpMaxDataRetransmissions", "SynAttackProtect", "TcpNumConnections", "TcpTimedWaitDelay", "EnableCompoundTcp", "MaxUserPort", "MaxFreeTcbs", "MaxHashTableSize", "DisableTaskOffload" -ErrorAction SilentlyContinue
            netsh int tcp set global autotuninglevel=disabled | Out-Null
            
            Set-ItemProperty "HKCU:\Control Panel\Accessibility\Keyboard Response" -Name "Flags" -Value "126" -ErrorAction SilentlyContinue

            Remove-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Kernel" -Name "GlobalTimerResolutionRequests" -ErrorAction SilentlyContinue
            bcdedit /deletevalue disabledynamictick 2>&1 | Out-Null
            
            [uint32]$currentRes = 0
            [NativeMethods]::NtSetTimerResolution(5000, $false, [ref]$currentRes) | Out-Null

            schtasks /delete /tn "NONGKIM_TimerResolution" /f 2>&1 | Out-Null
        }
        
        $lblStatus.Text = "SUCCESS"; $lblStatus.ForeColor = [System.Drawing.Color]::LimeGreen
    }
    catch {
        $ShortError =$_.Exception.Message
        if ($ShortError.Length -gt 35) { $ShortError =$ShortError.Substring(0,35) + "..." }
        $lblStatus.Text = "FAILED: $ShortError"; $lblStatus.ForeColor = [System.Drawing.Color]::Crimson
    }
    finally {
        $btnRun.Enabled =$true; $rbApply.Enabled =$true; $rbReset.Enabled =$true
        $ErrorActionPreference =$OldErrorActionPreference
    }
})

[void]$form.ShowDialog()
