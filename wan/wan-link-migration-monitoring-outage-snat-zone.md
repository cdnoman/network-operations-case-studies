# WAN Link Migration Case Study: Monitoring Outage Caused by Missing SNAT Source Zone

> Note: This case study is sanitized for public sharing. Organization names, real IP addresses, hostnames, screenshots, and sensitive topology details have been removed, changed, or generalized.

## 1. Overview

A dedicated WAN internet link was migrated from an existing firewall path directly to a next-generation firewall HA cluster.

The objective was to remove an intermediate firewall hop from the WAN path and free that firewall for Disaster Recovery infrastructure use.

After the migration, selected users had internet access successfully. However, the monitoring platform started showing multiple other WAN links as down.

Those WAN links were not actually down. The issue was caused by the monitoring VM losing its own internet path because its source zone was missing from the SNAT policy.

## 2. Environment

The environment included:

- Dedicated WAN internet link
- WAN switch
- Existing firewall
- Next-generation firewall HA cluster
- Internet access gateway
- Monitoring platform VM
- Users zone
- Trusted VM zone
- SNAT policy
- PBR policy
- Security policy
- Production monitoring system
- Multiple WAN links

## 3. Original Traffic Path

Before migration, the dedicated WAN link followed this path:

```text
ISP WAN Link
   ↓
WAN Switch
   ↓
Existing Firewall
   ↓
Next-Generation Firewall
   ↓
Selected Users / Dependent Services
```

The existing firewall was acting as an intermediate hop in the WAN path.

## 4. New Traffic Path

After migration, the WAN link was directly landed on the next-generation firewall HA cluster:

```text
ISP WAN Link
   ↓
WAN Switch
   ↓
Next-Generation Firewall HA Cluster
   ↓
Selected Users / Dependent Services
```

The intermediate firewall was removed from this specific WAN path.

## 5. Change Objective

The main objective of the migration was:

- Remove the intermediate firewall hop
- Land the WAN link directly on the next-generation firewall HA cluster
- Keep the same dedicated internet access for selected high-authority users
- Free the existing firewall for Disaster Recovery infrastructure use
- Maintain production internet access without service impact

## 6. Change Implemented

The following configurations were created on the next-generation firewall:

- SNAT policy
- PBR policy
- Security policy
- WAN interface connectivity
- Access control for selected users

Initial testing showed that selected users were able to access the internet through the migrated WAN link.

At this stage, the migration appeared successful.

## 7. Problem Observed

After the migration, the monitoring platform started showing multiple other WAN links as down.

This created confusion because:

- The newly migrated WAN link was working
- Selected users had internet access
- The other WAN links were not physically landed on the same firewall path
- The other WAN links were terminating through a different internet access gateway
- But the monitoring platform still showed them as down

A strange behavior was observed:

```text
New WAN link enabled  → Other WAN links showed DOWN in monitoring
New WAN link disabled → Other WAN links showed UP in monitoring
```

This pattern indicated that the WAN links were most likely not actually down.

The issue was related to the monitoring path.

## 8. Initial Assumptions

At first, the issue appeared to be related to one of the following:

- WAN routing issue
- PBR mismatch
- Firewall policy issue
- Monitoring platform polling issue
- Internet access gateway issue
- Upstream path problem
- Asymmetric routing
- SNAT issue
- Return path issue
- Monitoring reachability problem

Further troubleshooting was required to identify the actual root cause.

## 9. Troubleshooting Process

The troubleshooting process started by validating the new WAN link and then checking the monitoring system behavior.

### 9.1 User Internet Testing

Selected users were tested through the migrated WAN link.

Result:

```text
Selected users internet access: Working
New WAN link: Working
PBR for selected users: Working
Security policy: Working
```

This confirmed that the basic user traffic path was working.

### 9.2 Monitoring Platform Status

The monitoring platform was showing other WAN links as down.

However, those WAN links were not part of the migrated firewall path.

This meant the issue was not necessarily with the other WAN links themselves.

The monitoring platform’s own connectivity needed to be checked.

### 9.3 Monitoring VM Internet Testing

The monitoring VM was then checked.

It was discovered that the monitoring VM had also been using the same dedicated WAN link for internet access.

This dependency was missed during the migration.

### 9.4 Path Testing

When the monitoring VM was shifted to another WAN path, internet access was restored for the monitoring VM.

As soon as the monitoring VM regained internet access, the other WAN links also started showing as UP in the monitoring platform.

This confirmed that the WAN links were not actually down.

The monitoring system had lost its own working internet path.

## 10. Root Cause

The SNAT policy on the next-generation firewall included only the Users zone.

The Trusted VM zone, where the monitoring VM was located, was not included in the SNAT policy.

Because of this:

- Selected users were able to access the internet
- The monitoring VM could not access the internet properly through the migrated WAN path
- Monitoring polling failed
- Other WAN links appeared down in the monitoring platform

The WAN links were healthy.

The monitoring VM’s internet path was broken.

## 11. Root Cause Summary

```text
SNAT policy included:
✔ Users zone

SNAT policy missed:
✘ Trusted VM zone
✘ Monitoring VM
```

The actual issue was not with the WAN links.

The monitoring VM internet path was broken because its source zone was missing from SNAT.

## 12. Fix Applied

The Trusted VM zone was added to the SNAT policy.

After adding the Trusted VM zone:

```text
Monitoring VM internet: Restored
Monitoring platform polling: Restored
Other WAN links status: UP
Selected users internet: Working
Migrated WAN link: Working
```

The issue was resolved immediately.

## 13. Verification After Fix

After the fix, the following verification was performed:

- Confirmed selected users still had internet access
- Confirmed monitoring VM had internet access
- Confirmed monitoring platform polling was restored
- Confirmed other WAN links showed UP
- Checked SNAT policy source zones
- Checked PBR path for users and monitoring VM
- Checked security policy hit count
- Verified that traffic was following the expected WAN path
- Confirmed the migrated WAN link remained stable

## 14. Final Result

The WAN link migration was completed successfully.

The intermediate firewall was removed from the path and became available for Disaster Recovery infrastructure use.

The dedicated WAN link was directly landed on the next-generation firewall HA cluster.

Selected users continued using the dedicated internet link.

The monitoring platform was restored after including the Trusted VM zone in the SNAT policy.

## 15. Technical Explanation

This issue happened because the migration was tested mainly from the user traffic perspective.

Selected users were working, so the migration initially looked successful.

However, the monitoring VM was also dependent on the same WAN link.

Because the monitoring VM belonged to a different source zone, it did not match the SNAT policy that was created for the Users zone.

In firewall migrations, traffic does not only depend on interface status or routing.

Traffic also depends on:

- Source zone
- Destination zone
- SNAT policy
- PBR policy
- Security policy
- Return path
- Service dependency mapping

If one dependent source zone is missed, some services can fail even when user internet appears normal.

## 16. Useful Checks

During similar WAN/firewall migration issues, the following checks are useful:

```text
Check WAN interface status
Check default route
Check PBR policy
Check SNAT policy
Check source zone matching
Check security policy hit count
Check monitoring VM internet access
Check DNS resolution from monitoring VM
Check traceroute from monitoring VM
Check traffic logs
Check NAT logs
Check return path
```

For network/firewall troubleshooting, useful validation questions are:

```text
Is the source zone correct?
Is the destination zone correct?
Is SNAT matching the source?
Is PBR matching the correct source/user/subnet?
Is the security policy allowing the traffic?
Is the monitoring system also dependent on this link?
Is return traffic following the expected path?
```

## 17. Prevention Recommendations

To avoid similar issues in future migrations:

- Prepare a dependency list before migration
- Identify all users, servers, VMs, and monitoring tools using the WAN link
- Validate all source zones before creating SNAT policies
- Include monitoring systems in the migration test plan
- Test from user endpoints and server/VM zones
- Verify PBR, SNAT, and security policies together
- Confirm monitoring platform reachability after migration
- Document rollback steps before production changes
- Do not close the change only after user testing
- Validate end-to-end service health

## 18. Key Lessons

- A migration is not complete only because user traffic works.
- Monitoring systems must be included in migration testing.
- SNAT, PBR, and security policies must include all dependent source zones.
- VM zones should be reviewed during WAN/firewall migrations.
- False monitoring alerts can happen when the monitoring system loses its own connectivity.
- Always validate users, servers, VMs, monitoring tools, and return paths before closing a change.
- Dependency mapping is critical before production migration.
- A working WAN interface does not guarantee every dependent service is working.
- User testing and monitoring testing are both required.

## 19. One-Line Takeaway

The WAN links were not down. The monitoring path was broken because the monitoring VM source zone was missing from SNAT.
