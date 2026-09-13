# Enterprise IT Help Desk Lab

A hands-on Windows enterprise support environment built to practice and document real-world IT troubleshooting, Active Directory administration, Group Policy, DNS, SMB file sharing, printer deployment, authentication, and employee onboarding.

This project contains 10 documented support incidents performed in a virtualized Windows domain environment. Each incident follows a troubleshooting workflow: identify symptoms, gather evidence, isolate the cause, implement a resolution, and validate the result.

## Lab Environment

| Component | Configuration |
|---|---|
| Virtualization | Oracle VirtualBox |
| Domain | hoodie.local |
| Domain Controller | DC01 |
| Client Workstation | CLIENT01 |
| Server OS | Windows Server 2022 |
| Client OS | Windows 11 Pro |
| Directory Services | Active Directory Domain Services |
| DNS | Windows DNS |
| File Services | SMB |
| Policy Management | Group Policy |
| Print Services | Windows Print Services |
| Administration | PowerShell and Windows administrative tools |

## Architecture

```text
                    hoodie.local
                         |
                        DC01
                  192.168.10.10
                         |
          +--------------+--------------+
          |              |              |
   Active Directory     DNS       File / Print
          |              |          Services
          +--------------+--------------+
                         |
                 VirtualBox Network
                         |
                      CLIENT01
                  192.168.10.20
                         |
                  Domain Users
                   /        \
                 IT      Accounting
                            |
                    +-------+-------+
                    |               |
              Accounting Z:    Shared Printer
                    |               |
           \\DC01\Accounting  \\DC01\Accounting-Printer
```

## Incident Portfolio

| Incident | Scenario | Technologies / Skills |
|---|---|---|
| [INC-001](incidents/INC-001/) | Command Prompt Disabled by Group Policy | GPO, gpresult, PowerShell, policy troubleshooting |
| [INC-002](incidents/INC-002/) | Domain User Account Lockout | Active Directory, account lockout policy, ADUC |
| [INC-003](incidents/INC-003/) | Accounting SMB Share Unavailable | SMB, NTFS permissions, security groups, DNS |
| [INC-004](incidents/INC-004/) | Internal Intranet DNS and Web Service | DNS, IIS, nslookup, packet analysis |
| [INC-005](incidents/INC-005/) | Accounting Mapped Drive Unavailable | Group Policy Preferences, SMB, drive mapping |
| [INC-006](incidents/INC-006/) | Internal Resources Unavailable Due to DNS Misconfiguration | DNS troubleshooting, PowerShell, GPO validation |
| [INC-007](incidents/INC-007/) | Domain User Cannot Sign In | Active Directory, account status, authentication |
| [INC-008](incidents/INC-008/) | Windows Time Synchronization and Kerberos Validation | W32Time, Kerberos, klist, SMB authentication |
| [INC-009](incidents/INC-009/) | Accounting Network Printer Deployment | Print Server, printer drivers, PowerShell, GPO |
| [INC-010](incidents/INC-010/) | New Accounting Employee Onboarding | Active Directory, security groups, GPO, SMB, printers |

## Selected Technical Work

### Active Directory Administration

Built and administered a Windows domain environment using Active Directory Domain Services. Tasks included creating users, organizing accounts with OUs, managing security-group membership, troubleshooting locked and disabled accounts, and provisioning new employees.

### Group Policy

Used Group Policy and Group Policy Preferences to configure and troubleshoot user settings and automatically deploy departmental resources.

Examples included:

- Troubleshooting a command prompt restriction with gpresult.
- Configuring account lockout behavior.
- Mapping the Accounting Z: drive.
- Deploying the Accounting shared printer.
- Validating policy application from CLIENT01.

Common validation commands included:

```powershell
gpupdate /force
gpresult /r
```

### File Services and Permissions

Created and administered the Accounting SMB share:

```text
\\DC01\Accounting
```

Used the `Accounting_Users` security group for department-based authorization and Group Policy Preferences to map the resource as Z:.

Access was validated from CLIENT01 by successfully creating files on the departmental share.

### DNS and Internal Services

Configured and troubleshot Windows DNS for domain and internal-resource resolution.

Testing included DNS record creation, client DNS misconfiguration, hostname resolution, domain-controller connectivity, IIS availability, and packet-level investigation of DNS behavior.

Tools included:

```text
nslookup
ping
ipconfig
PowerShell
packet capture
```

### Windows Print Services

Built a simulated departmental printer environment on DC01 and investigated printer queue, driver, sharing, and deployment issues.

The final shared queue was:

```text
\\DC01\Accounting-Printer
```

The printer was deployed automatically to Accounting users through Group Policy and validated from CLIENT01.

### Authentication and Windows Time

Investigated Windows domain time synchronization and Kerberos behavior using:

```text
w32tm
klist
PowerShell
```

Significant clock skew was intentionally introduced between CLIENT01 and DC01. The experiment did not reproduce a Kerberos outage, so the documented findings reflect the observed authentication behavior rather than assuming a failure.

## Capstone: New Employee Onboarding

INC-010 combined multiple components of the lab into an end-to-end onboarding workflow for a new Accounting employee.

The workflow included:

1. Creating the user in Active Directory.
2. Placing the account in the Accounting OU.
3. Requiring a password change at first logon.
4. Assigning membership in Accounting_Users.
5. Authenticating the employee from CLIENT01.
6. Automatically mapping Z: to \\DC01\Accounting.
7. Automatically deploying \\DC01\Accounting-Printer.
8. Confirming the appropriate Accounting GPOs applied.
9. Validating write access to the departmental share.

This demonstrated the relationship between identity, authentication, group-based authorization, Group Policy, and enterprise resource access.

## Troubleshooting Methodology

Incidents were approached using a repeatable support process:

```text
Ticket / Symptom
      |
      v
Establish Baseline
      |
      v
Gather Evidence
      |
      v
Form Hypothesis
      |
      v
Test and Isolate
      |
      v
Implement Resolution
      |
      v
Validate User Access
      |
      v
Document Findings
```

The project also documents tests that did not behave as originally expected. Troubleshooting conclusions were based on observed evidence rather than changing the findings to match the initial hypothesis.

## Skills Demonstrated

- Active Directory Domain Services
- Active Directory Users and Computers
- User and account administration
- Organizational Units
- Security groups
- Group-based authorization
- Group Policy
- Group Policy Preferences
- Windows DNS
- SMB file sharing
- NTFS permissions
- Windows Print Services
- IIS
- Kerberos fundamentals
- Windows Time
- PowerShell
- Windows command-line administration
- Network troubleshooting
- Authentication troubleshooting
- Access-control troubleshooting
- Employee onboarding
- Root cause analysis
- Technical documentation
- Evidence collection and validation

## Repository Structure

```text
enterprise-it-support-lab/
|
├── README.md
└── incidents/
    ├── INC-001/
    ├── INC-002/
    ├── INC-003/
    ├── INC-004/
    ├── INC-005/
    ├── INC-006/
    ├── INC-007/
    ├── INC-008/
    ├── INC-009/
    └── INC-010/
```

Each incident directory contains a README describing the issue, investigation, resolution, validation, and skills demonstrated. Available screenshots and supporting evidence are stored with the corresponding incident.

## Project Purpose

This lab was built to develop practical experience relevant to entry-level IT support, help desk, desktop support, and junior systems administration roles.

The project focuses on complete user-impacting scenarios rather than isolated commands. Each incident required investigation of the environment, troubleshooting of the underlying issue, implementation of a resolution or controlled test, and validation of the resulting user or system state.

The goal is to demonstrate practical Windows enterprise support skills and a repeatable, evidence-based troubleshooting process.
