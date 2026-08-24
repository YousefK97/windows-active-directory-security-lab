# Active Directory Security Lab — Findings Report

## 1. Executive Summary

A controlled Windows Active Directory environment was deployed using a Windows Server Domain Controller and a Windows 10 domain client.

The investigation and validation focused on Active Directory organization, account management, Group Policy security controls, Windows Firewall configuration, Remote Desktop access, PowerShell administration, and domain connectivity.

The lab successfully demonstrated centralized management of Windows security settings through Active Directory and Group Policy.

## 2. Environment

**Domain Controller:** `DC01`

**Domain:** `corp.local`

**NetBIOS Domain:** `CORP`

**Domain Controller IP:** `192.168.87.10`

**Windows Client:** `MSEDGEWIN10`

**Virtualization:** VMware

## 3. Active Directory Findings

The following departmental organizational units were successfully configured:

* IT
* HR
* Sales
* Management

Four domain users were created and placed into their appropriate organizational units.

| User         | Department | Status  |
| ------------ | ---------- | ------- |
| John Smith   | IT         | Enabled |
| Maria Brown  | HR         | Enabled |
| Alex Taylor  | Sales      | Enabled |
| David Wilson | Management | Enabled |

Four corresponding security groups were configured:

| Security Group | Assigned User |
| -------------- | ------------- |
| IT-Admins      | John Smith    |
| HR-Users       | Maria Brown   |
| Sales-Users    | Alex Taylor   |
| Management     | David Wilson  |

PowerShell validation confirmed the expected group memberships and user OU placement.

## 4. Security Control Findings

### Account Lockout

The `Domain Account Security Policy` GPO was configured with:

* Threshold: 5 failed attempts
* Lockout duration: 15 minutes
* Counter reset: 15 minutes

This provides a basic protection against repeated password-guessing attempts.

### Windows Firewall

The `IT Firewall Policy` GPO configured Windows Defender Firewall on the Windows 10 client.

The client was verified with:

* Firewall enabled
* Inbound connections blocked
* Outbound connections allowed

This provides a basic host-based network security control.

### Remote Desktop

Remote Desktop Services access was configured through Group Policy.

The `CORP\IT-Admins` security group was added to the Windows 10 client's `Remote Desktop Users` group.

This demonstrates group-based authorization rather than granting access to individual users.

## 5. Domain and Network Findings

The Windows 10 client was successfully joined to `corp.local`.

DNS resolution successfully identified:

`DC01.corp.local → 192.168.87.10`

Domain Controller discovery using `nltest` successfully identified `DC01` as the Domain Controller for the domain.

Group Policy results confirmed that the client received multiple domain policies, including:

* Default Domain Policy
* Domain Account Security Policy
* IT Firewall Policy
* IT Security Policy

## 6. PowerShell Findings

PowerShell was used to validate the Active Directory environment.

The following activities were successfully performed:

* Domain user enumeration
* Security group enumeration
* Group membership validation
* Computer account validation
* Organizational Unit enumeration
* User Distinguished Name validation

The PowerShell results matched the expected Active Directory design.

## 7. Investigation Limitations

The Windows Security Event Log was examined as part of the lab.

Authentication-related events were present, including computer-account and network authentication activity. However, a clean user authentication event suitable for final evidence was not isolated.

Rather than altering audit settings solely to produce a specific event, the event-log investigation was left as a limitation of the current lab.

This is documented as an investigation limitation rather than being presented as a successful finding.

## 8. Privileged Access Review

A review of privileged Active Directory groups was performed using PowerShell.

The following results were identified:

| Privileged Group | Membership |
|---|---|
| Domain Admins | Administrator |
| Enterprise Admins | Administrator |
| Administrators | Domain Admins, Enterprise Admins, Administrator |
| IT-Admins | John Smith |

The review confirmed that the four departmental users were not members of the highly privileged `Domain Admins`, `Enterprise Admins`, or `Administrators` groups.

John Smith was assigned to the custom `IT-Admins` security group to provide IT-specific administrative access without adding the account directly to the highest-privilege domain groups.

### Security Finding

**Finding:** Privileged access is separated from normal departmental accounts.

**Risk:** Excessive membership in privileged groups could allow unnecessary administrative access and increase the impact of a compromised account.

**Current Control:** Administrative access is restricted through dedicated security groups and the highest-privilege groups contain only the built-in Administrator account.

**Assessment:** The current configuration demonstrates a basic least-privilege approach. Further hardening could include dedicated administrative accounts, stronger controls around the built-in Administrator account, and regular privileged-group membership reviews.

## 9. Security Assessment

The lab demonstrates a basic but functional Active Directory security baseline.

Positive controls identified:

* Centralized account lockout policy
* Host-based Windows Firewall
* Group-based Remote Desktop authorization
* Separation of users into departmental OUs
* Security groups for access management
* Centralized Group Policy management
* PowerShell-based administrative validation

Potential improvements include:

* Advanced Windows auditing
* Centralized event collection
* Security Information and Event Management (SIEM) integration
* Stronger password policies
* Least-privilege administration
* Additional endpoint security policies
* Regular review of privileged group membership

## 10. Conclusion

The Active Directory lab successfully demonstrated practical Windows enterprise administration and security concepts in a controlled environment.

The project provides hands-on evidence of experience with Active Directory, Group Policy, Windows Firewall, access control, PowerShell, DNS, and Windows client/domain administration.
