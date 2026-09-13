# INC-010 - New Accounting Employee Onboarding

## Ticket Summary

HR requested IT provisioning for a new Accounting employee, Maya Chen. The objective was to create the employee's domain identity, provide appropriate department authorization, enforce a first-logon password change, and validate access to standard Accounting resources from CLIENT01.

## Environment

- Domain: hoodie.local
- Domain Controller: DC01
- Client: CLIENT01
- New employee: Maya Chen
- Username: mchen
- Department: Accounting
- Security group: Accounting_Users
- Department share: \\DC01\Accounting
- Mapped drive: Z:
- Department printer: \\DC01\Accounting-Printer
- Drive GPO: Accounting Drive Mapping
- Printer GPO: Accounting Printer Deployment

## Provisioning

A new Active Directory user account was created for Maya Chen in the Accounting OU.

User logon name:

    mchen

The account was configured to require:

    User must change password at next logon

This allowed an initial temporary lab password to be replaced during the employee's first interactive domain sign-in.

## Authorization

Maya was added to:

    Accounting_Users

This followed a group-based authorization model rather than assigning departmental resource permissions directly to the individual user.

The access model used in the lab was:

    User Account
        |
        v
    Accounting_Users
        |
        v
    Accounting Resources

## First Logon

Maya signed into CLIENT01 using:

    HOODIE\mchen

Windows required the password to be changed at first logon as expected.

After the password change, Windows created Maya's local user profile and loaded the desktop successfully.

## Group Policy Validation

Group Policy Result was reviewed on CLIENT01.

The user was identified as:

    HOODIE\mchen

The distinguished name showed Maya in the Accounting OU:

    CN=Maya Chen,OU=Accounting,DC=hoodie,DC=local

The session also showed membership in:

    Accounting_Users

Accounting-related Group Policy results included:

    Accounting Drive Mapping
    Accounting Printer Deployment

This confirmed that the Accounting policies were being applied to the newly provisioned employee.

## Mapped Drive Validation

PowerShell was used to inspect filesystem drives:

    Get-PSDrive -PSProvider FileSystem

The result showed:

    Z:    \\DC01\Accounting

This confirmed automatic deployment of the Accounting departmental drive.

## Printer Validation

PowerShell was used to inspect installed printers:

    Get-Printer | Select-Object Name,Type,ComputerName

The result included:

    \\DC01\Accounting-Printer    Connection    DC01

This confirmed automatic deployment of the Accounting network printer.

## Resource Access Test

A file was created through the mapped Accounting drive:

    "INC-010 Maya Chen access validated $(Get-Date)" | Out-File "Z:\INC-010-Maya-Access-Test.txt"

The file was then queried with:

    Get-Item "Z:\INC-010-Maya-Access-Test.txt" | Select-Object Name,Length,LastWriteTime

The command returned:

    INC-010-Maya-Access-Test.txt

This validated that Maya had write access to the Accounting departmental share, rather than merely having the drive mapped.

## Resolution

The new Accounting employee was successfully provisioned and validated.

Completed onboarding tasks included:

- Created the domain user in the Accounting OU.
- Required a password change at first logon.
- Added the user to Accounting_Users.
- Successfully authenticated the user on CLIENT01.
- Verified creation of the user's Windows profile.
- Verified Accounting Group Policy application.
- Verified automatic Z: drive mapping to \\DC01\Accounting.
- Verified automatic deployment of \\DC01\Accounting-Printer.
- Successfully created a test file on the departmental share.

## Validation Summary

Identity:
    HOODIE\mchen

Organizational Unit:
    Accounting

Security Group:
    Accounting_Users

Mapped Resource:
    Z: -> \\DC01\Accounting

Network Printer:
    \\DC01\Accounting-Printer

Applied Accounting Policies:
    Accounting Drive Mapping
    Accounting Printer Deployment

Write Access:
    Successfully created INC-010-Maya-Access-Test.txt

## Skills Demonstrated

- Active Directory user provisioning
- Organizational Unit administration
- Temporary password and first-logon workflow
- Security group administration
- Group-based authorization
- Windows domain authentication
- Group Policy validation
- Group Policy Preferences
- Network drive deployment
- Shared printer deployment
- SMB share access validation
- PowerShell verification
- End-to-end employee onboarding

## Status

Resolved
