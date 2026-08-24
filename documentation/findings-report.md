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

### Endpoint Privilege Review

The local `Administrators` group on the Windows 10 domain client (`MSEDGEWIN10`) was reviewed using PowerShell.

The following members were identified:

| Account / Group             | Source           |
| --------------------------- | ---------------- |
| `CORP\Domain Admins`        | Active Directory |
| `MSEDGEWIN10\Administrator` | Local            |
| `MSEDGEWIN10\IEUser`        | Local            |

`CORP\Domain Admins` provides domain-level administrators with administrative access to the workstation, which is a common default configuration for domain-joined Windows systems.

The local `IEUser` account was also identified as a member of the local Administrators group. This represents a potential least-privilege concern because a local user account with administrative privileges has greater ability to modify the workstation and security configuration.

### Security Observation

**Finding:** `MSEDGEWIN10\IEUser` has local administrator privileges.

**Risk:** A compromised local account with administrative privileges could have greater control over the endpoint and potentially weaken security controls.

**Recommendation:** In a production environment, local administrative privileges should be restricted to accounts that require them for legitimate administrative tasks. Standard user accounts should normally operate without local administrator privileges.

**Lab Decision:** The account was not removed because this is a controlled lab environment and the existing configuration was preserved for documentation and analysis.

## 9. Password Policy Assessment

The default domain password policy was reviewed using PowerShell.

### Initial Configuration

The initial configuration was:

| Setting                 | Initial Value |
| ----------------------- | ------------: |
| Minimum password length |  7 characters |
| Password complexity     |       Enabled |
| Maximum password age    |       42 days |
| Minimum password age    |         1 day |
| Password history        |  24 passwords |

The minimum password length of seven characters was identified as a potential weakness because longer passwords and passphrases provide greater resistance to password-guessing attacks.

### Remediation

The minimum password length was increased from **7 to 12 characters** using the Active Directory default domain password policy.

The updated configuration was then verified using PowerShell.

### Security Finding

**Finding:** The original minimum password length was only seven characters.

**Risk:** Shorter passwords can reduce resistance to password-guessing and brute-force attacks.

**Remediation:** Increased the domain minimum password length to 12 characters.

**Result:** The updated policy was successfully verified.

Other password-policy controls, including complexity and password history, were already enabled.

**Evidence:** `37-password-policy-hardening.png`

## 10. Advanced Audit Policy Assessment

The Windows Advanced Audit Policy configuration was reviewed on the Domain Controller before making any changes.

The initial review identified that Process Creation auditing was not enabled.

### Initial Audit Configuration

| Audit Category            | Initial Configuration |
| ------------------------- | --------------------- |
| Logon                     | Success and Failure   |
| Account Lockout           | Success               |
| User Account Management   | Success               |
| Security Group Management | Success               |
| Process Creation          | No Auditing           |

### Remediation

Process Creation auditing was enabled for both successful and failed events using:

```powershell
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
```

The configuration was then verified using `auditpol`.

### Validation

A controlled `notepad.exe` process was launched on the Domain Controller.

Windows subsequently generated **Security Event ID 4688**, confirming that the Process Creation audit policy was functioning.

The event was retrieved using PowerShell and confirmed to contain:

* Event ID: 4688
* Provider: Microsoft-Windows-Security-Auditing
* Process creation information
* Event timestamp

### Security Finding

**Finding:** Process Creation auditing was initially disabled.

**Risk:** Without process-creation auditing, investigators have less visibility into programs executed on the Windows system.

**Remediation:** Enabled Process Creation auditing.

**Result:** Event ID 4688 was successfully generated and verified after controlled process execution.

**Evidence:**

* `38-process-creation-auditing.png`
* `39-process-creation-event-4688.png`

## 11. Security Assessment

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

## 12. Conclusion

The Active Directory lab successfully demonstrated practical Windows enterprise administration and security concepts in a controlled environment.

The project provides hands-on evidence of experience with Active Directory, Group Policy, Windows Firewall, access control, PowerShell, DNS, and Windows client/domain administration.
