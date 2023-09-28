---
layout: post
title: Active Directory Enumeration
titledate: 09/26/23
tags: ["infosec", "offsec", "AD", "powershell", "powerview", "sharphound", "bloodhound"]
---

#### legacy tools

for this sceanario we assume we have gained access to the AD via a client

    $ xfreerdp /u:jim /d:corp.com /v:192.168.1.1

enumerate users

    C: net user /domain

net user to diisplay users in domain

    C: net user admin /domain

net group to display groups in domain

    C: net group /domain

display members in specific group

    C: net group "Protected" /domain

#### powershell & .NET

in order to communicate with the AD we need to use the LDAP path

    - LDAP://HostName[:PortNumber][/DistinguishedName]

we can write a script to help us generate this first let's get the hostname for the PDC

    PS> [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

let's create a domain variable and print it to the screen for the script

```powershell
$domainObj [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
$domainObj
```

to run the script we need to bypass the execution policy

    PS> powershell -ep bypass

if we improve upon the script to get us the PDC role owner name and the distinguished name needed for the LDAP path it will look like

```powershell
$PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
$DN = ([adsi]'').distinguishedName 
$LDAP = "LDAP://$PDC/$DN"
$direntry = New-Object System.DirectoryServices.DirectoryEntry($LDAP)

$dirsearcher = New-Object System.DirectoryServices.DirectorySearcher($direntry)
$dirsearcher.filter="samAccountType=805306368"
$result = $dirsearcher.FindAll()

Foreach($obj in $result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
    }

    Write-Host "-------------------------------"
}
```

let's convert the script to a function that accepts user input

```powershell
function LDAPSearch {
    param (
        [string]$LDAPQuery
    )

    $PDC = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().PdcRoleOwner.Name
    $DistinguishedName = ([adsi]'').distinguishedName

    $DirectoryEntry = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$PDC/$DistinguishedName")

    $DirectorySearcher = New-Object System.DirectoryServices.DirectorySearcher($DirectoryEntry, $LDAPQuery)

    return $DirectorySearcher.FindAll()

}
```

    PS> Import-Module .\function.ps1

now we can query specific sam account types

    PS> LDAPSearch -LDAPQuery "(samAccountType=805306368)"

we can search a specific objclass

    PS> LDAPSearch -LDAPQuery "(objectclass=group)"

if our results contain a lot of groups we can iterate through them

    PS > foreach ($group in $(LDAPSearch -LDAPQuery "(objectCategory=group)")) {$group.properties | select {$_.cn},{$_.member}}

we can specify a specific department of a group

    PS> $sales = LDAPSearch -LDAPQuery "(&(objectCategory=group)(cn=Sales Department))"

list the member attribute

    PS> $sales.properties.member

our results show the Development Dept is a member of the Sales Dept so we can enumerate that

    PS> $group = LDAPSearch -LDAPQuery "(&(objectCategory=group)(cn=Development Department*))"

    PS> $group.properties.member

if there is another nested group like Management we can enumerate that

    PS> $group = LDAPSearch -LDAPQuery "(&(objectCategory=group)(cn=Management Department*))"

    PS> $group.properties.member
        - CN=jen,CN=Users,DC=corp,DC=com
    
#### powerview

    PS> Import-Module .\PowerView.ps1

domain info

    PS> Get-NetDomain

get users

    PS> Get-NetUser

list domain groups

    PS> Get-NetGroups

using pipe to display more info

    PS> Get-NetUser | select cn,pwdlastset,lastlogon

    PS> Get-NetGroup "Sales Department" | select member

#### enumerating OS

in powerview

    PS> Get-NetComputer

    PS> Get-NetComputer | select operatingsystem,dnshostname

#### user permissions

sometimes we may have gained credentials or access to a certain account or system but it may be fruitful to be patient and see all the information we can access with the current account we have first - we want to exhuast all options for each path before moving on

powerview has a command that lets us check if the current user has admin privs on any system in the domain

    PS> Find-LocalAdminAccess

we can check logged on users 

    PS> Get-NetSession -ComputerName files04

if no output is received a user may not be loged on, but we can verify that with the verbose flag

    PS> Get-NetSession -ComputerName files04 -Vebose

check windows OS

    PS> Get-NetComputer | select dnshostname,operatingsystem,operatingsystemversion

#### psloggedon

PsLoggedOn from SysInternals will enumerate registry keys under HKEY_USERS to retrieve the security identifiers logged-in users and convert them to usernames

    PS> .\PsLoggedon.exe \\files04

#### service accounts

services launched by the system run as a Service Account

use setspn to enumerate the iis_service user

    PS> setspn -L iis_service

powerview can also enumerate spns

    PS> Get-NetUser -SPN | select samaccountname,serviceprincipalname

if we find a web server we can try to see the ip it resolves to

    PS> nslookup.exe web04.corp.com

#### object permission

    PS> Find-InterestingDomainAcl | select identityreferencename,activedirectoryrights,acetype,objectdn | ?{$_.IdentityReferenceName -NotContains "DnsAdmins"} | ft

### NOTE: can use above to find genericall for user and then change perms with - net user john pass123! \domain

after this we can log in as the user and run 

    PS> Find-LocalAdminAccess

to find if we have local admin access then we can move to that machine to see what more we can uncover

enumerate ACLS for object group inside powerview

    PS> Get-ObjectAcl -Identity user

    PS> Get-ObjectAcl -Identity "Group  Department" | ? {$_.ActiveDirectoryRights -eq "GenericAll"} | select SecurityIdentifier,ActiveDirectoryRights

convert SIDs results to readable ouptput

    PS> "S-1-5-21-1987370270-658905905-1781884369-512","S-1-5-21-1987370270-658905905-1781884369-1104","S-1-5-32-548","S-1-5-18","S-1-5-21-1987370270-658905905-1781884369-519" | Convert-SidToName

add ourselves to a specific group

    PS> net group "Management Department" myuser /add /domain

check that we were added

    PS> Get-NetGroup "Group Department" | select member

delete member

    PS> net group "Group Department" myuser /del /domain

### NOTE: GenericAll is a powerful ACL permission to have tied to an account

#### enumerating domain shares

powerview has this capability

    PS> Find-DomainShare

now we need to dig into the returned shares - let's look at sysvol as it will have relevant domain info

    PS> ls \\dc1.corp.com\sysvol\corp.com\

### NOTE: investigate every folder we discover in search of interesting items

we find a gpp encrypted password that we can decrypt on our attacking machine

    $ gpp-decrypt "+bsY0V3d4/KgX3VJdO/vyepPfAN1zMFTiQDApgR92JE"

- make sure to investigate all shares! 

    PS> ls \\FILES04\docshare

#### sharphound

sharphound can be compiled used as an exe or a powershell script

    PS > Import-Module .\Sharphound.ps1

invoke script

    PS> Get-Help Invoke-BloodHound

    PS> Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\Users\user\Desktop\ -OutputPrefix "corp audit"

this will store collected domain data to the user desktop in a zip format

#### bloodhound

now we can run bloodhound on it

first we need to start Neo4j

    $ sudo neo4j start

the service will be available at 

- http://localhost:7474

    user: neo4j 
    pass: neo4j 

leave Neo4j running and start bloodhound

    $ bloodhound

now upload our sharphound zip info

we can view analysis results 

we can view relevant domain admin info

    - Find all Domain Admins

as well as the shortest path to the domain admin in the analysis tab

if we have admin access to a machine we can inform bloodhound by marking it as owned s principal with a skull icon, this will change the relevant route information for what it recommends as an attack path

in our example session user1 had acess to machine 1 but bloodhound has informed us that user1 also has domain access to machine2 where user2 has domain admin creds, so if we can get his creds from machine 2 we can gain domain admin access - which we may have not known before

