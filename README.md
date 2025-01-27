# check_vmware_datastores
nagios check using VMware PowerCLI for snapshot health

# Requirements
PowerShell for Linux and VMware PowerCLI installed on nagios server

# Installation Steps

Install PowerShell on Linux
```
yum install https://github.com/PowerShell/PowerShell/releases/download/v7.4.6/powershell-7.4.6-1.rh.x86_64.rpm
```

Add VMware PowerCLI from the PowerShell Gallery
```
[root@linuxbox]# pwsh
PowerShell 7.4.6
PS /home/someuser> Install-Module -Name VMware.PowerCLI -Scope AllUsers
```

Disable the PowerCLI call-home to VMware for usage statistics reporting.
```
[root@linuxbox]# pwsh
PowerShell 7.4.6
PS> Set-PowerCLIConfiguration -Scope AllUsers -ParticipateInCEIP $false
```

# Sample output

If everything is ok, output will look similar to:
```
VMware snapshots OK total_snapshots:7  largest_single_snapshot:12 GB , total_size:23 GB , oldest_snapshot:4 days
```

If there are problems, each problem will be displayed in the nagios check output.
Please note that if there are many problems with old snapshots, the output message may be very long.  
To make the following example easier to read, each problem has been printed on a separate line.
```
VMware snapshots WARN, 5 issues found, please cleanup snapshots.   
'dbserv' snapshot     'VM Snapshot 1/23/2025, 8:24:12 PM' is 12 GB in size, max is 10 GB. 
'dbserv' snapshot     'VM Snapshot 1/23/2025, 7:16:25 PM' is 4 days old, which exceeds max age of 3 days. 
'dbserv' snapshot     'VM Snapshot 1/23/2025, 8:24:12 PM' is 4 days old, which exceeds max age of 3 days. 
'fileserver' snapshot 'VM Snapshot 1/23/2025, 7:17:35 PM' is 4 days old, which exceeds max age of 3 days. 
'dbserv' has 5 snapshots, max is 3.
 total_snapshots:4  largest_single_snapshot:12 GB , total_size:23 GB , oldest_snapshot:4 days
```

`
