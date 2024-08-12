---
title: "ACL Enumeration"
date: 2024-12-8
draft: false
tags: ["Active-Directory", "access control list"]
description: "ACL Enumeration"
---

![](https://www.shutterstock.com/image-photo/acl-access-control-list-permissions-associated-2193882835)

## ACL Enumeration

Enumerating ACLs using powerview and getting a graphical representation using BloodHound.

### Powerview
Powerview can be used to enumerate ACLs, but the task of going through all the results is extremely time-consuming and likely inaccurate, a good example is when running *Find-InterestingDomainACL* we you will receive a ton of information back that you'd need to go through o make sense of.

```
PS C:\htb> Find-InterestingDomainAcl

ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : ab721a53-1e2f-11d0-9819-00aa0040529b
AceFlags                : ContainerInherit
AceType                 : AccessAllowedObject
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-3842939050-3880317879-2865463114-5189
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security 
                          Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group

ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : 00299570-246d-11d0-a768-00aa006e0529
AceFlags                : ContainerInherit
AceType                 : AccessAllowedObject
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-3842939050-3880317879-2865463114-5189
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security 
                          Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group
```
Going through such data during a time-boxed assessment, you'll never get through it all or find anything interesting before the assessment is over, but there is a better way to utilise PowerView by performing targeted enumeration starting off with user that we have control over.

### Case scenario
We have user called *wesley* we can use this user to find interesting ACL rights that we could take advantage of, first off we need to get the SID of our target user to search effectively.

```
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> $sid = Convert-NameToSid wley
```
Later on we can then use the *Get-DomainObjectACL* function to perform our targeted search, the function we aregoing to be using it's purpose is to find all domain objects that our user has rights over by mapping the user's SID using *$sid* variable to the *SecurityIdentifier* property which is what tells us who has the given righ over an object.

*N|B* 
|One important thing to note is that if we search without the flag ResolveGUIDs, we will see results like the below, where the right ExtendedRight does not give us a clear picture of what ACE entry the user wley has over damundsen. This is because the ObjectAceType property is returning a GUID value that is not human readable.

```
PS C:\htb> Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}

ObjectDN               : CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114-1176
ActiveDirectoryRights  : ExtendedRight
ObjectAceFlags         : ObjectAceTypePresent
ObjectAceType          : 00299570-246d-11d0-a768-00aa006e0529
InheritedObjectAceType : 00000000-0000-0000-0000-000000000000
BinaryLength           : 56
AceQualifier           : AccessAllowed
IsCallback             : False
OpaqueLength           : 0
AccessMask             : 256
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1181
AceType                : AccessAllowedObject
AceFlags               : ContainerInherit
IsInherited            : False
InheritanceFlags       : ContainerInherit
PropagationFlags       : None
AuditFlags             : None
```

A good thing to note here is upon enumeration of the *ObjectAceType* you would come to find our that the user has rights to force change other user's passwords [https://learn.microsoft.com/en-us/windows/win32/adschema/r-user-force-change-password](read..more)
