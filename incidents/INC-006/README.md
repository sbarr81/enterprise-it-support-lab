# INC-006 - Internal Resources Unavailable Due to DNS Misconfiguration

## Ticket Information

- Incident ID: INC-006
- Priority: Medium
- User: Jordan Lee
- Department: Accounting
- Workstation: CLIENT01
- Domain: hoodie.local
- Category: Network Connectivity / DNS
- Status: Resolved

## User Report

Jordan Lee reported being unable to access internal company resources, including the Accounting drive and internal intranet.

Other Accounting users were reported to be working normally.

## Environment

- Domain: hoodie.local
- Domain Controller / DNS Server: DC01
- DC01 IP Address: 192.168.10.10
- Client: CLIENT01
- CLIENT01 IP Address: 192.168.10.20
- Subnet Mask: 255.255.255.0
- Intranet: intranet.hoodie.local
- Accounting Share: \\DC01\Accounting
- Accounting Mapped Drive: Z:

## Troubleshooting

### 1. Reviewed CLIENT01 Network Configuration

Ran:

ipconfig /all

Reviewed the workstation's IPv4 address, subnet mask, default gateway, and DNS configuration.

CLIENT01 used a static IPv4 configuration on the 192.168.10.0/24 lab network.

During troubleshooting, the configured DNS server was identified as:

192.168.10.99

The expected domain DNS server was:

192.168.10.10

Because internal Active Directory resources depend on domain DNS, the incorrect DNS server was treated as a primary suspect.

### 2. Tested Basic IP Connectivity

Tested connectivity from CLIENT01 to DC01 by IP address:

ping 192.168.10.10

The ping succeeded.

This demonstrated that CLIENT01 could reach DC01 at the network layer and indicated that the issue was not a basic loss of IP connectivity.

### 3. Tested DNS Resolution

Ran:

nslookup intranet.hoodie.local

The DNS request timed out while CLIENT01 was configured to use 192.168.10.99.

This supported the suspected DNS client configuration problem.

### 4. Corrected the DNS Server

Changed CLIENT01's DNS server from:

192.168.10.99

to the correct domain DNS server:

192.168.10.10

PowerShell command used:

Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "192.168.10.10"

Cleared the DNS client cache:

Clear-DnsClientCache

Verified the configured IPv4 DNS server with:

Get-DnsClientServerAddress -InterfaceAlias "Ethernet" -AddressFamily IPv4

The configured DNS server was confirmed as:

192.168.10.10

### 5. Validated Internal Web Service Connectivity

Tested the intranet HTTP service:

Test-NetConnection intranet.hoodie.local -Port 80

Result:

TcpTestSucceeded : True

Opened:

http://intranet.hoodie.local

The intranet loaded successfully.

This confirmed successful user-facing access to the internal web resource after correcting the DNS configuration.

### 6. Validated Accounting Share Access

Opened the Accounting share directly:

\\DC01\Accounting

The share opened successfully.

This confirmed that the underlying SMB resource remained accessible.

### 7. Investigated Accounting Drive Mapping

During validation, net use did not display a mapped drive entry.

Because INC-005 had deployed the Accounting drive through Group Policy Preferences, the Group Policy state was investigated rather than manually recreating the mapping.

Generated a Group Policy Results report:

gpresult /h "$env:USERPROFILE\Desktop\gpresult.html"

The report confirmed:

- Jordan Lee was located in the Accounting OU
- Accounting Drive Mapping was an applied GPO
- Group Policy Drive Maps reported Success
- Z: was configured through the Accounting Drive Mapping GPO
- Action was Update
- Location was \\DC01\Accounting
- Reconnect was enabled
- Label was Accounting

No changes were made to the GPO because the policy configuration and processing results were valid.

### 8. Verified Z: in the User Session

Checked filesystem drives with:

Get-PSDrive -PSProvider FileSystem

Z: appeared with the root:

\\DC01\Accounting

Validated the drive path:

Test-Path Z:\

Result:

True

Ran:

Get-PSDrive Z

This again confirmed that Z: mapped to:

\\DC01\Accounting

This demonstrated that the Accounting drive was available in Jordan Lee's user session despite the earlier net use observation.

### 9. Performed Post-Reboot Validation

After CLIENT01 was restarted, the DNS configuration and internal resources were tested again.

Verified:

- DNS remained configured as 192.168.10.10
- Z: remained accessible
- intranet.hoodie.local loaded successfully

This confirmed that the DNS correction persisted and internal resource access remained functional after reboot.

## Root Cause

CLIENT01 was configured to use an incorrect DNS server address:

192.168.10.99

instead of the hoodie.local domain DNS server:

192.168.10.10

The incorrect DNS configuration caused DNS queries for internal domain resources to fail.

## Resolution

Restored CLIENT01's IPv4 DNS server configuration to:

192.168.10.10

Then cleared the local DNS client cache and validated access to internal services.

No changes to Active Directory permissions, SMB permissions, IIS, or the Accounting Drive Mapping GPO were required.

## Validation

The resolution was validated by confirming:

- CLIENT01 DNS server was 192.168.10.10
- DC01 remained reachable
- TCP port 80 to intranet.hoodie.local succeeded
- http://intranet.hoodie.local loaded successfully
- \\DC01\Accounting opened successfully
- Z: resolved to \\DC01\Accounting
- Test-Path Z:\ returned True
- DNS and internal resource access remained functional after CLIENT01 reboot

## Diagnostic Note

nslookup displayed timeout behavior during portions of testing.

After the DNS configuration was corrected, Windows successfully resolved and accessed hostname-dependent internal resources, including intranet.hoodie.local. Because functional resolver and application-layer tests succeeded, the nslookup behavior was treated as a lab-specific diagnostic anomaly rather than evidence that the repaired DNS configuration remained nonfunctional.

## Evidence

### 01_client_network_baseline.png

Shows CLIENT01 network configuration from the lab environment.

This screenshot represents the healthy baseline configuration and does not show the temporary 192.168.10.99 DNS fault.

### 02_drive_mapping_gpo_result.png

Shows the Group Policy Results report confirming successful Drive Maps processing and application of the Accounting Drive Mapping GPO.

### 03_drive_mapping_gpo_details.png

Shows the Z: Group Policy Preference configuration, including the \\DC01\Accounting location and successful policy result.

### 04_accounting_drive_validation.png

Shows Get-PSDrive and Test-Path validation confirming Z: was accessible and mapped to \\DC01\Accounting.

## Evidence Limitation

No screenshot was captured showing CLIENT01 configured with the temporary 192.168.10.99 DNS server or the initial failed nslookup test.

These steps were performed during troubleshooting but are not represented as photographic evidence in this incident record.

## Skills Demonstrated

- Windows network troubleshooting
- TCP/IP configuration analysis
- DNS troubleshooting
- PowerShell network administration
- DNS client configuration
- DNS cache management
- Layered troubleshooting methodology
- ICMP connectivity testing
- HTTP service validation
- SMB resource validation
- Active Directory Group Policy analysis
- Group Policy Preferences troubleshooting
- PowerShell drive validation
- Root cause isolation
- Post-reboot validation
- Technical incident documentation

## Outcome

INC-006 was resolved successfully.

Troubleshooting isolated the primary issue to an incorrect DNS server configured on CLIENT01. The workstation's DNS configuration was restored to the hoodie.local domain DNS server, and access to internal resources was validated.

The investigation also demonstrated the importance of validating reported symptoms independently and avoiding unnecessary changes to functioning services, permissions, or Group Policy configuration.
