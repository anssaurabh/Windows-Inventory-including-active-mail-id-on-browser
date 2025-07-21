# Get-AssetDetails.ps1
$report = @()

# Hostname and user
$hostname = $env:COMPUTERNAME
$username = $env:USERNAME

# MAC Address
$mac = (Get-NetAdapter | Where-Object {$_.Status -eq "Up"} | Select-Object -First 1).MacAddress

# System Info
$computerSystem = Get-CimInstance Win32_ComputerSystem
$bios = Get-CimInstance Win32_BIOS
$cpu = Get-CimInstance Win32_Processor | Select-Object -First 1
$os = Get-CimInstance Win32_OperatingSystem
$gpu = Get-CimInstance Win32_VideoController

# Storage
$storage = Get-CimInstance Win32_LogicalDisk -Filter "DriveType=3" |
    Measure-Object -Property Size -Sum
$totalStorageGB = [math]::Round($storage.Sum / 1GB, 2)

# Office 365 check
$o365 = Get-ItemProperty "HKLM:\Software\Microsoft\Office\ClickToRun\Configuration" -ErrorAction SilentlyContinue
$officeStatus = if ($o365) { "Installed (Office 365)" } else { "Not Installed" }

# Antivirus check
$av = Get-CimInstance -Namespace "root\SecurityCenter2" -ClassName AntivirusProduct -ErrorAction SilentlyContinue
$avStatus = if ($av) { $av.displayName } else { "No Antivirus Detected" }

# Report
$report += "===== IT ASSET REPORT ====="
$report += "Hostname            : $hostname"
$report += "User                : $hostname\$username"
$report += "MAC Address         : $mac"
$report += "Device Type         : Notebook"
$report += "Model               : $($computerSystem.Model)"
$report += "Serial Number       : $($bios.SerialNumber)"
$report += "CPU                 : $($cpu.Name)"
$report += "RAM (GB)            : $([math]::Round($computerSystem.TotalPhysicalMemory / 1GB))"
$report += "Storage (GB)        : $totalStorageGB"
$report += "GPU                 : $($gpu.Name)"
$report += "OS                  : $($os.Caption) $($os.OSArchitecture)"
$report += "Office 365          : $officeStatus"
$report += "Antivirus           : $avStatus"
$report += "`nReport Generated On: $(Get-Date)"

# Output to file
$report | Out-File -FilePath ".\Asset_Report_$hostname.txt" -Encoding UTF8

# Optional: Open the report automatically
notepad ".\Asset_Report_$hostname.txt"
