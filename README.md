Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing

Add-Type -TypeDefinition @"
using System;
using System.Runtime.InteropServices;
public class DPI {
    [DllImport("user32.dll")]
    public static extern bool SetProcessDPIAware();
}
"@
[DPI]::SetProcessDPIAware()

$form = New-Object System.Windows.Forms.Form
$form.Text = "NONGKIM"
$form.Size = New-Object System.Drawing.Size(500, 500)
$form.StartPosition = 'CenterScreen'
$form.BackColor = [System.Drawing.Color]::Black
$form.FormBorderStyle = 'FixedDialog'
$form.MaximizeBox = $false

$fontTitle = New-Object System.Drawing.Font("Impact", 40, [System.Drawing.FontStyle]::Regular)
$fontNormal = New-Object System.Drawing.Font("Consolas", 12, [System.Drawing.FontStyle]::Regular)

$lblTitle = New-Object System.Windows.Forms.Label
$lblTitle.Text = "NONGKIM"
$lblTitle.Font = $fontTitle
$lblTitle.ForeColor = [System.Drawing.Color]::White
$lblTitle.Size = New-Object System.Drawing.Size(300, 70)
$lblTitle.Location = New-Object System.Drawing.Point(100, 40)
$lblTitle.TextAlign = 'MiddleCenter'
$form.Controls.Add($lblTitle)

$lblSub = New-Object System.Windows.Forms.Label
$lblSub.Text = "SETTING"
$lblSub.Font = $fontNormal
$lblSub.ForeColor = [System.Drawing.Color]::DimGray
$lblSub.Size = New-Object System.Drawing.Size(300, 25)
$lblSub.Location = New-Object System.Drawing.Point(100, 110)
$lblSub.TextAlign = 'MiddleCenter'
$form.Controls.Add($lblSub)

$rbApply = New-Object System.Windows.Forms.RadioButton
$rbApply.Text = "APPLY"
$rbApply.Location = New-Object System.Drawing.Point(170, 150)
$rbApply.Size = New-Object System.Drawing.Size(80, 30)
$rbApply.ForeColor = [System.Drawing.Color]::White
$rbApply.Checked = $true
$form.Controls.Add($rbApply)

$rbReset = New-Object System.Windows.Forms.RadioButton
$rbReset.Text = "RESET"
$rbReset.Location = New-Object System.Drawing.Point(260, 150)
$rbReset.Size = New-Object System.Drawing.Size(80, 30)
$rbReset.ForeColor = [System.Drawing.Color]::White
$form.Controls.Add($rbReset)

$txtKey = New-Object System.Windows.Forms.TextBox
$txtKey.Location = New-Object System.Drawing.Point(150, 200)
$txtKey.Size = New-Object System.Drawing.Size(200, 30)
$txtKey.PasswordChar = '*'
$txtKey.BackColor = [System.Drawing.Color]::FromArgb(20, 20, 20)
$txtKey.ForeColor = [System.Drawing.Color]::White
$txtKey.TextAlign = 'Center'
$form.Controls.Add($txtKey)

$btnRun = New-Object System.Windows.Forms.Button
$btnRun.Text = "ENTER"
$btnRun.Location = New-Object System.Drawing.Point(150, 250)
$btnRun.Size = New-Object System.Drawing.Size(200, 40)
$btnRun.BackColor = [System.Drawing.Color]::Black
$btnRun.ForeColor = [System.Drawing.Color]::White
$btnRun.FlatStyle = 'Flat'
$form.Controls.Add($btnRun)

$lblStatus = New-Object System.Windows.Forms.Label
$lblStatus.Size = New-Object System.Drawing.Size(400, 30)
$lblStatus.Location = New-Object System.Drawing.Point(50, 320)
$lblStatus.TextAlign = 'MiddleCenter'
$lblStatus.ForeColor = [System.Drawing.Color]::Gray
$form.Controls.Add($lblStatus)

$btnRun.Add_Click({
    if ($txtKey.Text -ne "NK@7788") {
        $lblStatus.Text = "ACCESS DENIED"; $lblStatus.ForeColor = [System.Drawing.Color]::DarkRed; return
    }
    $txtKey.Enabled = $false; $btnRun.Enabled = $false; $rbApply.Enabled = $false; $rbReset.Enabled = $false

    $OldErrorActionPreference = $ErrorActionPreference
    $ErrorActionPreference = "Stop"

    try {
        for ($i = 1; $i -le 100; $i++) {
            $lblStatus.Text = "PROCESSING... $i%"; [System.Windows.Forms.Application]::DoEvents(); Start-Sleep -Milliseconds 40
            if ($i -eq 50) {
                if ($rbApply.Checked) {
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
                    if ($Adapter) { Set-NetAdapterAdvancedProperty -Name $Adapter.Name -DisplayName "Interrupt Moderation" -DisplayValue "Disabled" -ErrorAction SilentlyContinue; Set-NetAdapterAdvancedProperty -Name $Adapter.Name -DisplayName "Flow Control" -DisplayValue "Disabled" -ErrorAction SilentlyContinue }
                    
                    New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\AFD\Parameters" -Name "DefaultReceiveWindow" -Value 65535 -PropertyType DWord -Force | Out-Null
                    New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\AFD\Parameters" -Name "DefaultSendWindow" -Value 65535 -PropertyType DWord -Force | Out-Null
                    New-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" -Name "NetworkThrottlingIndex" -Value 1 -PropertyType DWord -Force | Out-Null
                    New-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile" -Name "SystemResponsiveness" -Value 0 -PropertyType DWord -Force | Out-Null
                    New-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks\Games" -Name "GPU Priority" -Value 8 -PropertyType DWord -Force | Out-Null
                    New-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks\Games" -Name "Priority" -Value 6 -PropertyType DWord -Force | Out-Null
                    New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\PriorityControl" -Name "Win32PrioritySeparation" -Value 38 -PropertyType DWord -Force | Out-Null
                    New-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" -Name "HwSchMode" -Value 2 -PropertyType DWord -Force | Out-Null
                    
                    Set-ItemProperty "HKCU:\Control Panel\Mouse" -Name "MouseSpeed" -Value 0
                    Set-ItemProperty "HKCU:\Control Panel\Mouse" -Name "MouseThreshold1" -Value 0
                    Set-ItemProperty "HKCU:\Control Panel\Mouse" -Name "MouseThreshold2" -Value 0
                    
                    Set-ItemProperty "HKCU:\Control Panel\Accessibility\Keyboard Response" -Name "AutoRepeatDelay" -Value 250
                    Set-ItemProperty "HKCU:\Control Panel\Accessibility\Keyboard Response" -Name "AutoRepeatRate" -Value 12
                    
                    foreach ($Svc in @("DiagTrack", "dmwappushservice", "SysMain", "MapsBroker", "Fax")) { Set-Service $Svc -StartupType Disabled -ErrorAction SilentlyContinue; Stop-Service $Svc -Force -ErrorAction SilentlyContinue }
                    Remove-Item -Path "$env:TEMP\*", "C:\Windows\Temp\*" -Recurse -Force -ErrorAction SilentlyContinue; [System.GC]::Collect()

                    $InterfacesPath = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces"
                    Get-ChildItem -Path $InterfacesPath | ForEach-Object {
                        $SubPath = $_.Name -replace "HKEY_LOCAL_MACHINE", "HKLM:"
                        New-ItemProperty -Path $SubPath -Name "TcpAckFrequency" -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
                        New-ItemProperty -Path $SubPath -Name "TcpNoDelay" -Value 1 -PropertyType DWord -Force -ErrorAction SilentlyContinue | Out-Null
                    }
                    netsh int tcp set global autotuninglevel=normal | Out-Null
                    netsh int tcp set global congestionprovider=cubic | Out-Null
                    netsh int tcp set global dca=enabled | Out-Null
                    netsh int tcp set global chimney=enabled | Out-Null
                    netsh int tcp set global ecncapability=disabled | Out-Null
                    netsh int tcp set global timestamps=disabled | Out-Null

                    $TCPParameters = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters"
                    New-ItemProperty -Path $TCPParameters -Name "SackOpts" -Value 1 -PropertyType DWord -Force | Out-Null
                    New-ItemProperty -Path $TCPParameters -Name "Tcp1323Opts" -Value 1 -PropertyType DWord -Force | Out-Null
                    New-ItemProperty -Path $TCPParameters -Name "DefaultTTL" -Value 64 -PropertyType DWord -Force | Out-Null
                    
                    $QoSPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Psched"
                    if (!(Test-Path $QoSPath)) { New-Item -Path $QoSPath -Force | Out-Null }
                    New-ItemProperty -Path $QoSPath -Name "NonBestEffortLimit" -Value 0 -PropertyType DWord -Force | Out-Null

                    $SystemProfile = "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile"
                    New-ItemProperty -Path $SystemProfile -Name "NetworkThrottlingIndex" -Value 4294967295 -PropertyType DWord -Force | Out-Null
                } else {
                    $TCP = "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters"
                    Remove-ItemProperty $TCP -Name "TcpWindowSize", "GlobalMaxTcpWindowSize", "Tcp1323Opts", "DefaultTTL", "SackOpts", "TcpMaxDataRetransmissions", "SynAttackProtect", "TcpNumConnections", "TcpTimedWaitDelay", "EnableCompoundTcp", "MaxUserPort", "MaxFreeTcbs", "MaxHashTableSize", "DisableTaskOffload" -ErrorAction SilentlyContinue
                    netsh int tcp set global autotuninglevel=disabled | Out-Null
                }
            }
        }
        $lblStatus.Text = "SUCCESS"; $lblStatus.ForeColor = [System.Drawing.Color]::Green
    }
    catch {
        $txtKey.Enabled = $true; $btnRun.Enabled = $true; $rbApply.Enabled = $true; $rbReset.Enabled = $true
        $ShortError = $_.Exception.Message
        if ($ShortError.Length -gt 35) { $ShortError = $ShortError.Substring(0,35) + "..." }
        $lblStatus.Text = "FAILED: $ShortError"; $lblStatus.ForeColor = [System.Drawing.Color]::DarkRed
    }
    finally {
        $ErrorActionPreference = $OldErrorActionPreference
    }
})

[void]$form.ShowDialog()
