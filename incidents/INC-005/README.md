# INC-005 - Mapped Accounting Drive Unavailable

## Ticket Information

- Incident ID: INC-005
- Priority: Medium
- User: Jordan Lee
- Department: Accounting
- Workstation: CLIENT01
- Domain: hoodie.local
- Category: File Services / Access
- Status: Resolved

## User Report

Jordan Lee reported that the Accounting drive was no longer available on CLIENT01. The drive had previously been used to access files required for Accounting work. Other workstation functionality appeared normal.

Another Accounting user was reported to still have access to the Accounting resource.

## Environment

- Domain Controller: DC01
- Client: CLIENT01
- Domain: hoodie.local
- File Share: \\DC01\Accounting
- Mapped Drive: Z:
- Security Group: Accounting_Users
- Deployment Method: Group Policy Preferences

## Troubleshooting

### 1. Verified Active Directory User Placement

Confirmed Jordan Lee's Active Directory account:

- User: jlee
- OU: Accounting
- Domain: hoodie.local

This verified that the affected user was located in the expected Accounting organizational unit.

### 2. Verified Security Group Membership

Checked Jordan Lee's Active Directory group memberships.

Jordan was confirmed as a member of:

- Domain Users
- Accounting_Users

This indicated that the user had the expected Accounting security group membership.

### 3. Verified Accounting Folder Permissions

Reviewed the NTFS permissions on:

C:\Shares\Accounting

Confirmed that the Accounting_Users security group had Modify permissions on the folder.

No permission changes were made because the expected group-based access was already present.

### 4. Verified Network Connectivity

From CLIENT01, tested connectivity to DC01 using ping.

DC01 responded successfully, confirming basic network connectivity between the workstation and server.

### 5. Verified SMB Connectivity

From CLIENT01, tested TCP port 445 on DC01:

Test-NetConnection DC01 -Port 445

Result:

TcpTestSucceeded : True

This confirmed that CLIENT01 could reach the Windows SMB file-sharing service on DC01.

### 6. Tested the Share Directly

From CLIENT01, accessed:

\\DC01\Accounting

The share opened successfully.

This demonstrated that:

- DC01 was reachable
- SMB connectivity was functioning
- The Accounting share was available
- Jordan's permissions allowed access to the resource

The issue was therefore isolated from the underlying file share and permissions.

### 7. Checked Existing Drive Mappings

Ran:

net use

No mapped network drives were present in Jordan Lee's session.

This explained why the Accounting drive was unavailable even though the underlying UNC share remained accessible.

### 8. Reviewed Group Policy Deployment

Reviewed the existing Group Policy Objects on DC01.

No GPO existed for deploying the Accounting network drive.

A new GPO was created and linked to the Accounting OU:

Accounting Drive Mapping

The drive mapping was configured under:

User Configuration
Preferences
Windows Settings
Drive Maps

Configuration:

- Action: Update
- Location: \\DC01\Accounting
- Reconnect: Enabled
- Label: Accounting
- Drive Letter: Z:

### 9. Applied Group Policy

On CLIENT01, while logged in as Jordan Lee, ran:

gpupdate /force

After Group Policy refreshed successfully, the mapped drives were checked again with:

net use

The Accounting share was successfully mapped to Z:.

## Root Cause

Jordan Lee had valid permissions to the Accounting file share, and the SMB service and network path were functioning correctly.

The Accounting drive was unavailable because no mapped drive was present in Jordan's user session and no Group Policy Object existed to deploy the Accounting drive mapping.

## Resolution

Created and linked the Accounting Drive Mapping GPO to the Accounting OU.

The GPO deployed:

\\DC01\Accounting

as:

Z:

Group Policy was refreshed on CLIENT01, after which the Accounting mapped drive became available to Jordan Lee.

## Validation

Validated the resolution by:

- Confirming Z: appeared as a mapped network drive
- Opening the Accounting drive
- Creating INC-005-Access-Test.txt
- Writing data to the file
- Saving the file
- Reopening the file
- Successfully reading the saved contents

This confirmed successful end-to-end access to the Accounting file share through the mapped drive.

## Evidence

### 01_accounting_drive_mapping_success.png

Shows the successful Accounting network drive mapping after Group Policy deployment.

### 02_accounting_drive_access_validation.png

Shows successful access to the Accounting Z: drive and validation of the mapped resource.

## Skills Demonstrated

- Active Directory user administration
- Active Directory security group verification
- NTFS permission analysis
- Windows network troubleshooting
- SMB troubleshooting
- TCP port testing
- UNC path testing
- Group Policy Management
- Group Policy Preferences
- Enterprise drive mapping
- PowerShell troubleshooting
- Root cause isolation
- End-user access validation

## Outcome

INC-005 was resolved successfully.

The investigation demonstrated that the underlying file share, network connectivity, SMB service, and user permissions were operational. Troubleshooting isolated the issue to the absence of the mapped drive. A centrally managed Group Policy solution was implemented and validated successfully.
