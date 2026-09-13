# INC-002: Domain User Account Locked Out

## Ticket Information

- Category: Account / Authentication
- Priority: Medium
- User: Jordan Lee
- Department: Accounting
- Device: CLIENT01
- Domain: hoodie.local
- Status: Resolved

## Reported Issue

A domain user reported being unable to sign in to CLIENT01 after several unsuccessful password attempts.

The user stated that they had entered the wrong password multiple times and were subsequently unable to access their account.

## Initial Assessment

The repeated unsuccessful authentication attempts suggested that the domain account may have reached the configured account lockout threshold.

Before making changes to the user's account, I verified the account status in Active Directory.

## Troubleshooting

I opened Active Directory Users and Computers on DC01 and navigated to the Accounting organizational unit.

I located the Jordan Lee domain account and reviewed the Account properties.

The account showed that it was locked out.

I also verified the effective domain account lockout policy by running the net accounts command on DC01.

The effective domain policy showed:

- Account lockout threshold: 5 invalid logon attempts
- Account lockout duration: 30 minutes
- Account lockout observation window: 30 minutes

The user confirmed that they remembered their existing password.

Because the password was known, a password reset was not necessary.

## Root Cause

The domain account became locked after reaching the configured threshold of unsuccessful authentication attempts.

The user had entered an incorrect password multiple times, causing Active Directory to enforce the domain account lockout policy.

## Resolution

I opened the Jordan Lee account in Active Directory Users and Computers and used the Account settings to unlock the domain account.

I did not reset the user's password because the user confirmed that they knew the correct existing password.

This avoided making an unnecessary credential change.

## Validation

After the account was unlocked, the user attempted to sign in to CLIENT01 using the correct existing password.

Authentication completed successfully and the user regained access to the workstation.

## Skills Demonstrated

- Active Directory Users and Computers
- Domain user account administration
- Account lockout troubleshooting
- Authentication troubleshooting
- Windows Server administration
- Account lockout policy verification
- Least-change troubleshooting
- Root cause analysis
- End-user communication
- Post-resolution validation
- Technical documentation

## Outcome

The user's inability to sign in was traced to an Active Directory account lockout caused by repeated unsuccessful authentication attempts.

The account was unlocked without performing an unnecessary password reset.

The user successfully authenticated with the existing password and workstation access was restored.

Status: Resolved
