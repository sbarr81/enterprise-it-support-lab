# INC-007 - Domain User Cannot Sign In

## Ticket Information

- Incident ID: INC-007
- Priority: High
- User: Jordan Lee
- Department: Accounting
- Workstation: CLIENT01
- Domain: hoodie.local
- Category: Authentication / Active Directory
- Status: Resolved

## User Report

Jordan Lee reported being unable to sign in to CLIENT01 using the same domain credentials previously used successfully.

Other domain users were not reporting authentication problems, and no recent password change was documented.

## Environment

- Domain: hoodie.local
- Domain Controller: DC01
- Client Workstation: CLIENT01
- User: Jordan Lee
- Username: jlee
- User OU: Accounting
- Domain Account: HOODIE\jlee

## Troubleshooting

### 1. Reproduced the Authentication Problem

A fresh interactive domain sign-in was attempted on CLIENT01 using:

HOODIE\jlee

Windows reported that the account was disabled.

This provided a specific authentication error and shifted troubleshooting toward Jordan Lee's Active Directory account rather than immediately resetting the password.

### 2. Verified Domain Connectivity

Before modifying the account, CLIENT01's ability to communicate with the domain environment was checked.

Ran:

whoami

The session identified the user as:

hoodie\jlee

Checked the logon server:

echo $env:LOGONSERVER

Result:

\\DC01

Tested connectivity to the domain controller:

ping DC01

DC01 resolved to 192.168.10.10 and responded successfully with no packet loss.

These results demonstrated that CLIENT01 could communicate with DC01.

### 3. Verified the Computer Secure Channel

Ran:

Test-ComputerSecureChannel -Verbose

Result:

True

This confirmed that CLIENT01's secure channel with the hoodie.local domain was functioning.

Because domain connectivity and the workstation secure channel were healthy, no domain rejoin or computer-account repair was performed.

### 4. Investigated the User Account in Active Directory

On DC01, opened:

Server Manager
Tools
Active Directory Users and Computers

Navigated to:

hoodie.local
Accounting
Jordan Lee

Opened Jordan Lee's account properties and reviewed the Account settings.

The Account is disabled option was enabled.

This matched the error produced during the fresh interactive domain sign-in attempt.

### 5. Remediated the Account

In Jordan Lee's Active Directory account properties, cleared:

Account is disabled

Applied the change without modifying Jordan's password, OU placement, security group memberships, or other account properties.

### 6. Verified Active Directory Account Status

On DC01, ran:

Get-ADUser jlee -Properties Enabled |
Select-Object Name,Enabled

Result:

Jordan Lee
Enabled: True

This confirmed that the domain account had been successfully re-enabled.

### 7. Performed End-User Validation

Returned to CLIENT01 and performed a fresh domain sign-in using Jordan Lee's normal credentials.

The login completed successfully.

After login, verified the user context with:

whoami

Result:

hoodie\jlee

Verified the domain logon server with:

echo $env:LOGONSERVER

Result:

\\DC01

This confirmed successful end-to-end domain authentication after the account was re-enabled.

## Root Cause

Jordan Lee's Active Directory user account was disabled.

CLIENT01 retained healthy connectivity to DC01 and had a functioning domain secure channel. The authentication failure was therefore isolated to the state of Jordan Lee's user account rather than a workstation networking, domain trust, or password issue.

## Resolution

Re-enabled Jordan Lee's Active Directory account through Active Directory Users and Computers.

No password reset, domain rejoin, group membership change, or workstation configuration change was required.

## Validation

The resolution was validated by confirming:

- Jordan Lee's Active Directory account showed Enabled: True
- CLIENT01 maintained connectivity to DC01
- CLIENT01's domain secure channel returned True
- Jordan successfully completed a fresh domain sign-in
- whoami returned hoodie\jlee
- LOGONSERVER returned \\DC01

## Troubleshooting Note

An existing Jordan Lee session was initially accessible even after the account had been disabled.

Rather than treating that existing session as proof that the account was healthy, a complete sign-out was performed and a fresh interactive domain authentication attempt was made.

The fresh authentication attempt returned the disabled-account error.

This demonstrated the importance of reproducing authentication incidents with a fresh authentication attempt rather than relying solely on the state of an existing user session.

## Evidence

### 01_jlee_account_enabled.png

Shows Jordan Lee's Active Directory account after remediation with the account enabled.

### 02_jlee_login_validation.png

Shows successful post-remediation user-session validation on CLIENT01, including the authenticated domain user and domain logon server.

## Evidence Limitation

No screenshot was captured showing the Windows disabled-account error before remediation.

The error was observed and used during troubleshooting but is not represented as photographic evidence in this incident record.

## Skills Demonstrated

- Active Directory user administration
- Windows domain authentication troubleshooting
- Active Directory Users and Computers
- PowerShell Active Directory administration
- Domain controller connectivity testing
- Windows secure channel validation
- Authentication error analysis
- Root cause isolation
- Least-change remediation
- End-user login validation
- Incident documentation

## Outcome

INC-007 was resolved successfully.

Troubleshooting confirmed that CLIENT01 had healthy domain connectivity and a functioning secure channel before isolating the authentication failure to Jordan Lee's disabled Active Directory account.

The account was re-enabled without making unnecessary password, permission, group membership, or workstation changes. A fresh domain login succeeded and the authenticated user and logon server were validated.
