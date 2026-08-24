# Active Directory Security Lab — Lab Notes

## Lab Objective

The objective of this lab was to build a small Windows Active Directory environment and practice common Windows administration and security tasks in a controlled virtual environment.

The lab focused on:

* Deploying a Windows Server Domain Controller
* Creating an Active Directory domain
* Creating organizational units
* Creating domain users and security groups
* Joining a Windows 10 client to the domain
* Applying security policies through Group Policy
* Configuring Windows Defender Firewall
* Managing Remote Desktop access
* Using PowerShell to enumerate and validate Active Directory
* Verifying DNS and domain connectivity

## Lab Environment

### Domain Controller

* Hostname: `DC01`
* Operating System: Windows Server
* Domain: `corp.local`
* NetBIOS domain: `CORP`
* IP address: `192.168.87.10`

### Windows Client

* Hostname: `MSEDGEWIN10`
* Operating System: Windows 10
* Domain: `corp.local`
* Virtualization: VMware

## Active Directory Configuration

### Organizational Units

The following OUs were created:

* IT
* HR
* Sales
* Management

### Users

| User         | Username       | OU         |
| ------------ | -------------- | ---------- |
| John Smith   | `john.smith`   | IT         |
| Maria Brown  | `maria.brown`  | HR         |
| Alex Taylor  | `alex.taylor`  | Sales      |
| David Wilson | `david.wilson` | Management |

### Security Groups

| Group       | Member       |
| ----------- | ------------ |
| IT-Admins   | John Smith   |
| HR-Users    | Maria Brown  |
| Sales-Users | Alex Taylor  |
| Management  | David Wilson |

## Group Policy

### Account Lockout Policy

A domain-level GPO named `Domain Account Security Policy` was configured.

Settings:

* Account lockout threshold: 5 attempts
* Account lockout duration: 15 minutes
* Reset account lockout counter: 15 minutes

The policy was linked at the `corp.local` domain level.

### Firewall Policy

A GPO named `IT Firewall Policy` was configured to enable the Windows Defender Firewall.

The Windows 10 client was verified to have:

* Firewall enabled
* Inbound connections blocked
* Outbound connections allowed

### Remote Access Policy

A GPO named `IT Remote Access Policy` was created and configured to allow Remote Desktop Services connections.

The `CORP\IT-Admins` group was also added to the Windows 10 client's `Remote Desktop Users` group.

## PowerShell Validation

PowerShell was used to validate the Active Directory environment.

Commands used included:

```powershell
Get-ADUser -Filter *

Get-ADGroup -Filter *

Get-ADGroupMember -Identity "IT-Admins"

Get-ADComputer -Identity "MSEDGEWIN10"

Get-ADOrganizationalUnit -Filter *

Get-ADUser -Filter * | Select-Object Name,SamAccountName,DistinguishedName
```

These checks confirmed:

* Domain users existed and were enabled
* Security groups existed
* Group memberships were correct
* The Windows 10 computer account existed and was enabled
* The computer account was located in the IT OU
* The expected OUs existed
* Users were located in the correct departmental OUs

## Network and Domain Verification

DNS resolution was tested from the Windows 10 client:

```cmd
nslookup DC01.corp.local
```

The Domain Controller resolved to:

`192.168.87.10`

Domain Controller discovery was also tested:

```cmd
nltest /dsgetdc:corp.local
```

Group Policy application was verified with:

```cmd
gpresult /scope computer /r
```

The client successfully received several domain policies, including:

* Default Domain Policy
* Domain Account Security Policy
* IT Firewall Policy
* IT Security Policy

## Issues Encountered

Several troubleshooting issues were encountered during the lab.

### Group Policy Verification

The `gpresult` command initially produced access-related problems when run from a standard domain user account. Running the verification from an administrative session provided access to the computer-scope Group Policy results.

### Event Viewer

Attempts were made to identify authentication events in the Windows Security log. Several system and computer-account events were observed, but a clean user authentication event was not isolated for the final evidence set.

This demonstrated the importance of understanding where authentication events are generated and which system should be investigated during a domain authentication investigation.

### GPO Application

The `IT Remote Access Policy` was successfully created and linked at the domain level but did not appear in the client's applied Group Policy results during the lab verification. The policy was left in place without making unnecessary changes.

## Key Findings

The lab successfully demonstrated:

1. A functioning Active Directory domain.
2. Correct OU-based organization of users.
3. Security group-based access management.
4. Centralized account lockout configuration.
5. Centralized Windows Firewall configuration.
6. Remote Desktop access management.
7. PowerShell-based Active Directory administration.
8. DNS-based Domain Controller resolution.
9. Domain Controller discovery from the Windows client.
10. Successful application of multiple Group Policy objects.

## Conclusion

The lab provided practical experience with Windows Server, Active Directory, Group Policy, PowerShell, Windows Firewall, DNS, and Windows client administration.

The environment demonstrates how centralized Active Directory policies can be used to manage users, groups, security controls, and Windows endpoints within a controlled enterprise-style environment.
