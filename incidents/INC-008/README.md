# INC-008 - Windows Time Synchronization and Kerberos Validation

## Ticket Summary

This incident investigated the relationship between Windows domain time synchronization and Kerberos authentication.

CLIENT01 was intentionally configured with significant clock skew to simulate a common Active Directory authentication risk. Kerberos ticket behavior, Windows Time status, domain connectivity, and access to an authenticated SMB resource were then tested.

The lab did not reproduce a Kerberos authentication outage. Instead, the incident demonstrated the importance of validating observed behavior rather than assuming that clock skew automatically produces an authentication failure.

## Environment

- Domain: hoodie.local
- Domain Controller: DC01
- Client: CLIENT01
- Test user: Jordan Lee (jlee)
- Authentication protocol investigated: Kerberos
- Time service: Windows Time (W32Time)
- Authenticated resource: \\DC01\Accounting

## Baseline Validation

Kerberos tickets were inspected with:

    klist

The client had valid Kerberos tickets for the HOODIE.LOCAL domain.

Time synchronization between CLIENT01 and DC01 was measured with:

    w32tm /stripchart /computer:DC01 /dataonly /samples:5

The initial offset was only a few milliseconds, establishing a healthy baseline.

## Clock Skew Simulation

CLIENT01's system clock was intentionally moved approximately 10 minutes ahead:

    Set-Date -Date (Get-Date).AddMinutes(10)

A subsequent W32Time strip chart showed a difference of approximately 584 seconds between CLIENT01 and DC01.

The Windows Time service was later stopped on the client to preserve the intentionally introduced skew.

Further testing showed an even larger observed difference of approximately 1,133 seconds.

## Kerberos Testing

Existing Kerberos tickets were removed:

    klist purge

A new CIFS service ticket was then requested:

    klist get cifs/DC01.hoodie.local

Despite the intentional clock skew, the client successfully obtained a Kerberos service ticket in this lab environment.

Access to the Accounting SMB resource was also tested:

    Test-Path \\DC01\Accounting

The resource remained accessible.

## Findings

The experiment demonstrated significant client-to-domain-controller clock skew, but the expected Kerberos authentication failure was not reproduced.

It would therefore be inaccurate to document this incident as a Kerberos outage.

The observed results showed that:

- CLIENT01 and DC01 initially had closely synchronized clocks.
- Significant clock skew was successfully introduced.
- Windows Time behavior could be measured with W32Time.
- Kerberos tickets could be inspected and purged with Klist.
- A fresh CIFS Kerberos service ticket was still obtained during testing.
- The authenticated Accounting resource remained accessible.
- Time synchronization was subsequently restored.

## Resolution

CLIENT01's clock was returned to the correct time and time synchronization was restored.

After remediation, the client again showed a small time difference relative to DC01 and successfully obtained a fresh CIFS Kerberos ticket.

## Key Lesson

Clock synchronization is an important dependency in Active Directory environments, but troubleshooting conclusions must be based on observed evidence.

The lab intentionally created substantial clock skew, yet Windows authentication continued functioning during the test. Rather than reporting an authentication failure that did not occur, the incident was documented as a Windows Time and Kerberos behavior investigation.

## Skills Demonstrated

- Windows Time troubleshooting
- W32Time
- Active Directory time synchronization concepts
- Kerberos ticket inspection
- Klist
- Kerberos service ticket requests
- SMB authentication validation
- PowerShell system time administration
- Controlled fault injection
- Baseline comparison
- Evidence-based troubleshooting

## Status

Resolved / Investigation Complete
