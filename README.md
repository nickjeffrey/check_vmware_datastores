# check_vmware_snapshots
nagios check using VMware PowerCLI for snapshot health

# Requirements
PowerShell for Linux and VMware PowerCLI powershell module installed on Linux-based nagios server, low-privileged read-only user on vCenter.

# Install PowerShell on Linux (with internet access)

This section assumes that the Linux-based nagios server has access to the internet for downloading packages via HTTPS.

Install PowerShell on the Linux-based nagios server.

More details at https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-linux
```
which yum && yum install https://github.com/PowerShell/PowerShell/releases/download/v7.4.6/powershell-7.4.6-1.rh.x86_64.rpm
which apt && apt install https://github.com/PowerShell/PowerShell/releases/download/v7.4.7/powershell_7.4.7-1.deb_amd64.deb
```

# Install VMware.PowerCLI module on Linux (with internet access)

This section assumes that the Linux-based nagios server has access to the internet for downloading packages via HTTPS.

Confirm that the Linux-based nagios server has access to the PSGallery repository.
```
[root@linuxbox]# pwsh
PowerShell 7.4.6
PS> Get-PSRepository

Name          InstallationPolicy   SourceLocation
----          ------------------   --------------
PSGallery     Untrusted            https://www.powershellgallery.com/api/v2
```

If the PSGallery repository was not shown in the previous command, you will need to register that repository with the following commands.  Please note that internet access is required.
```
PS>  Register-PSRepository -Name 'PSGallery' -SourceLocation 'https://www.powershellgallery.com/api/v2/' -InstallationPolicy 'Trusted'
PS>  Get-PSRepository
```
Or if the above commands do not work, try these:
```
PS>  Register-PSRepository -Default
PS>  Get-PSRepository
```


Once you have confirmed that the PSGallery repository is available, add the VMware PowerCLI module from the PowerShell Gallery.  This command assumes the nagios server has internet access to obtain the packages.
```
[root@linuxbox]# pwsh
PowerShell 7.4.6
PS /home/someuser> Install-Module -Name VMware.PowerCLI -Scope AllUsers
```

Confirm that the VMware.PowerCLI module is installed:
```
PS /> Get-Module VMware.PowerCLI

ModuleType Version    PreRelease Name              ExportedCommands
---------- -------    ---------- ----              ----------------
Manifest   13.3.0.24…            VMware.PowerCLI
```


Disable the PowerCLI call-home to VMware for usage statistics reporting.
```
[root@linuxbox]# pwsh
PowerShell 7.4.6
PS> Set-PowerCLIConfiguration -Scope AllUsers -ParticipateInCEIP $false
```


# Install PowerShell on Linux (without internet access)

This section assumes that the Linux-based nagios server does not have access to the internet for downloading packages via HTTPS, so you will need to manually download packages on another system that does have internet access, and then copy those files to the Linux-based nagios server.

Install PowerShell on the Linux-based nagios server.

More details at https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-linux

In this example, download the appropriate RPM or DEB package to a machine that has internet access.  
```
cd c:\temp
curl -o powershell-7.4.6-1.rh.x86_64.rpm https://github.com/PowerShell/PowerShell/releases/download/v7.4.6/powershell-7.4.6-1.rh.x86_64.rpm
curl -o powershell_7.4.7-1.deb_amd64.deb https://github.com/PowerShell/PowerShell/releases/download/v7.4.7/powershell_7.4.7-1.deb_amd64.deb
```

Now that you have downloaded the appropriate RPM or DEB package, copy it over to the nagios server.
```
scp powershell* username@MyNagiosHost:/tmp
```

SSH into the nagios server and install the manually downloaded package.  Adjust package names as appropriate as newer versions are released.
```
which yum  && yum install /tmp/powershell-7.4.6-1.rh.x86_64.rpm
which dpkg && dpkg -i     /tmp/powershell_7.4.7-1.deb_amd64.deb
```

# Install VMware.PowerCLI module on Linux (without internet access)

This section assumes that the Linux-based nagios server does not have access to the internet for downloading packages via HTTPS, so you will need to manually download packages on another system that does have internet access, and then copy those files to the Linux-based nagios server.

On a machine that does have internet access, download the VMware.PowerCLI module to a temporary location.  This assumes you have access to the PSGallery repository.
```
PS> md C:\Temp\VMwarePowerCLI
PS> cd C:\Temp\VMwarePowerCLI
PS> Save-Module -Name VMware.PowerCLI -Path C:\Temp\VMwarePowerCLI
```

You will now see lots of subdirectories in the temporary location you downloaded the module and dependencies to:
```
PS C:\Temp\VMwarePowerCLI> ls


    Directory: C:\Temp\VMwarePowerCLI


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          2/7/2025  10:33 AM                VMware.CloudServices
d-----          2/7/2025  10:34 AM                VMware.DeployAutomation
d-----          2/7/2025  10:34 AM                VMware.ImageBuilder
d-----          2/7/2025  10:35 AM                VMware.PowerCLI
d-----          2/7/2025  10:34 AM                VMware.PowerCLI.Sdk
d-----          2/7/2025  10:34 AM                VMware.PowerCLI.Sdk.Types
d-----          2/7/2025  10:35 AM                VMware.PowerCLI.VCenter
d-----          2/7/2025  10:34 AM                VMware.PowerCLI.VCenter.Types.ApplianceService
d-----          2/7/2025  10:34 AM                VMware.PowerCLI.VCenter.Types.CertificateManagement
d-----          2/7/2025  10:35 AM                VMware.Sdk.Nsx.Policy
d-----          2/7/2025  10:34 AM                VMware.Sdk.Runtime
d-----          2/7/2025  10:35 AM                VMware.Sdk.Srm
d-----          2/7/2025  10:35 AM                VMware.Sdk.Vcf.CloudBuilder
d-----          2/7/2025  10:35 AM                VMware.Sdk.Vcf.SddcManager
d-----          2/7/2025  10:35 AM                VMware.Sdk.Vr
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance.Access
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance.Health
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance.InfraProfile
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance.LocalAccounts
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance.Logging
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance.Networking
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance.Recovery
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance.SupportBundle
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance.System
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance.Tls
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Appliance.Update
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Cis
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Cis.Tagging
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Content
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.ContentLibrary
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Esx.Hcl
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Esx.Hosts
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.Esx.Settings
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.SnapService
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.VAPI.Metadata
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Authentication
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Authorization
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.CertManagement
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.ConsumptionDomains
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Content
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Datastore
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Deployment
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Guest
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.HVC
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Identity
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Inventory
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.ISO
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.LCM
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.NamespaceManagement
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Namespaces
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.OVF
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Services
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Storage
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.SystemConfig
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Tagging
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Topology
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.TrustedInfrastructure
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.VCHA
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.Vm
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vCenter.VmTemplate
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphere.vStats
d-----          2/7/2025  10:34 AM                VMware.Sdk.vSphereRuntime
d-----          2/7/2025  10:33 AM                VMware.Vim
d-----          2/7/2025  10:33 AM                VMware.VimAutomation.Cis.Core
d-----          2/7/2025  10:34 AM                VMware.VimAutomation.Cloud
d-----          2/7/2025  10:33 AM                VMware.VimAutomation.Common
d-----          2/7/2025  10:33 AM                VMware.VimAutomation.Core
d-----          2/7/2025  10:34 AM                VMware.VimAutomation.Hcx
d-----          2/7/2025  10:33 AM                VMware.VimAutomation.License
d-----          2/7/2025  10:33 AM                VMware.VimAutomation.Nsxt
d-----          2/7/2025  10:33 AM                VMware.VimAutomation.Sdk
d-----          2/7/2025  10:34 AM                VMware.VimAutomation.Security
d-----          2/7/2025  10:33 AM                VMware.VimAutomation.Srm
d-----          2/7/2025  10:33 AM                VMware.VimAutomation.Storage
d-----          2/7/2025  10:34 AM                VMware.VimAutomation.StorageUtility
d-----          2/7/2025  10:33 AM                VMware.VimAutomation.Vds
d-----          2/7/2025  10:33 AM                VMware.VimAutomation.Vmc
d-----          2/7/2025  10:33 AM                VMware.VimAutomation.vROps
d-----          2/7/2025  10:34 AM                VMware.VimAutomation.WorkloadManagement
d-----          2/7/2025  10:34 AM                VMware.VumAutomation
```


Now that you have downloaded the packages using a machine with internet access, copy the files over to a temporary location on the nagios server.
```
scp -r c:\Temp\VMwarePowerCLI username@MyNagiosHost:/tmp
```

On the Linux-based nagios server, figure out the paths where PowerShell looks for installed modules
```
PS> $env:PSModulePath -split ':'
/root/.local/share/powershell/Modules
/usr/local/share/powershell/Modules
/opt/microsoft/powershell/7/Modules
```

In the above example, there are three locations that PowerShell looks for installed modules, we want the system-wide option, which is /usr/local/share/powershell/Modules.  Copy the files from the temporary location to the location that PowerShell expects the modules.
```
PS> Copy-Item -Path "/tmp/VMwarePowerCLI" -Destination /usr/local/share/powershell/Modules -Recurse
```


Now that the files have been copied to the expected location, you can import the module:
```
PS> Import-Module VMware.PowerCLI
```


Confirm that the VMware.PowerCLI module is installed:
```
PS> Get-Module VMware.PowerCLI

ModuleType Version    PreRelease Name              ExportedCommands
---------- -------    ---------- ----              ----------------
Manifest   13.3.0.24…            VMware.PowerCLI
```

Disable the PowerCLI call-home to VMware for usage statistics reporting.
```
[root@linuxbox]# pwsh
PowerShell 7.4.6
PS> Set-PowerCLIConfiguration -Scope AllUsers -ParticipateInCEIP $false
```



# vCenter configuration

## create vCenter userid

Create a read-only user account on the vCenter server.  

The nagios check will use these credentials to query vCenter for all the snapshots.

Login to vCenter web interface, click Administration menu, Users and Groups

<img src=images/adminmenu.png>




Set the domain to vsphere.local, click Add.

<img src=images/usersandgroups.png>




Enter the user details, click Add.

<img src=images/userdetails.png>

## assign read-only role to vCenter userid
Now we will assign a role to the nagios user so it has sufficient privilege to view snapshots.

Note that this only gives the userid on the vCenter server enough permission to view snapshots, but not to create or delete snapshots.

Click Global Permissions, Add.

Set the domain to vsphere.local

Set the user to nagios

Set the Role to Read-Only

Select the checkbox to Propagate to children  (to push the permission out to the ESXi hosts)

Click OK

<img src=images/changerole.png>


# Nagios configuration

Copy the check script and config file to the appropriate plugins folder and confirm the script is executable.  HINT: if you cannot run "git clone", manually copy and paste the file contents from the github page.
```
su - nagios
cd /tmp
git clone https://github.com/nickjeffrey/check_vmware_snapshots
cd check_vmware_snapshots
cp check_vmware_snapshots.cfg /usr/local/nagios/libexec/check_vmware_snapshots.cfg
cp check_vmware_snapshots     /usr/local/nagios/libexec/check_vmware_snapshots
chown nagios:nagios           /usr/local/nagios/libexec/check_vmware_snapshots.cfg
chown nagios:nagios           /usr/local/nagios/libexec/check_vmware_snapshots
chmod 744                     /usr/local/nagios/libexec/check_vmware_snapshots.cfg
chmod 755                     /usr/local/nagios/libexec/check_vmware_snapshots
```


Edit the default config file at /usr/local/nagios/libexec/check_vmware_snapshots.cfg, adjusting as appropriate for your environment.
```
# config file for check_vmware_snapshots script
# lines beginning with a hash mark # are ignored
# adjust the variables below to match your environment
vcenter=vcsa.example.com
username=nagios@vsphere.local
password=MySecretPassw0rd!
max_gb_per_snapshot=10
max_snapshots_per_vm=3
max_snapshot_age_days=7
```

Create a section similar to the following in the commands.cfg file on the nagios server.
```
# 'check_vmware_snapshots' command definition
define command {
      command_name    check_vmware_snapshots
      command_line    $USER1$/check_vmware_snapshots $ARG1$
      }

```

Create a section similar to the following in the services.cfg file on the nagios server.  This will use the default config file.
```
define service {
       use                             generic-service
       host_name                       vcenter.example.com
       service_description             VMware snapshots
       check_interval                  120              ; Actively check the host every 120 minutes
       check_command                   check_vmware_snapshots
       }
```


HINT: if you have multiple vCenter hosts, you can specify a custom config file for each stanza as shown below.
```
define service {
       use                             generic-service
       host_name                       vcenter1.example.com
       service_description             VMware snapshots
       check_interval                  120              ; Actively check the host every 120 minutes
       check_command                   check_vmware_snapshots!/usr/local/nagios/libexec/check_vmware_snapshots.vcenter1.cfg
       }

define service {
       use                             generic-service
       host_name                       vcenter2.example.com
       service_description             VMware snapshots
       check_interval                  120              ; Actively check the host every 120 minutes
       check_command                   check_vmware_snapshots!/usr/local/nagios/libexec/check_vmware_snapshots.vcenter2.cfg
       }

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
