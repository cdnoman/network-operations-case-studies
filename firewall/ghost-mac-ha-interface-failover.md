# Firewall HA Case Study: Ghost MAC Address Blocking HA Interface Failover

> Note: This case study is sanitized for public sharing. Organization names, real IP addresses, hostnames, interface names, screenshots, and sensitive topology details have been removed, changed, or generalized.

## 1. Overview

This case study documents a firewall High Availability troubleshooting scenario where HA behavior was affected due to a ghost or stale MAC address entry in the switching path.

The firewall HA pair was expected to provide failover between the primary and secondary firewall. The HA role change was working, but after failover, traffic behavior was not normal. The expected firewall unit became active, but some traffic was still not passing correctly.

The issue was later traced to a stale MAC address entry on the connected switching layer. Because of this, traffic was being forwarded toward the wrong Layer 2 path.

## 2. Environment

The environment included:

- Firewall HA pair
- Primary firewall
- Secondary firewall
- Core/distribution switch
- Firewall-connected switch ports
- Internal VLANs
- WAN/inside security zones
- HA heartbeat/interface monitoring
- Layer 2 MAC address learning
- Production traffic path

## 3. Expected HA Behavior

In a normal firewall HA setup:

- One firewall acts as the active/primary unit
- The second firewall remains standby/secondary
- If the active firewall fails, the standby firewall takes over
- Connected switches should relearn MAC addresses after failover
- Traffic should start forwarding toward the new active firewall path

Expected behavior:

```text
Primary Firewall Active
        ↓
Traffic forwards through Primary Firewall

Primary Firewall Failure
        ↓
Secondary Firewall Becomes Active
        ↓
Switch relearns MAC address
        ↓
Traffic forwards through Secondary Firewall
```

## 4. Problem Observed

During HA/failover testing, the firewall role change appeared successful, but traffic did not fully recover as expected.

Observed symptoms included:

- HA status looked healthy
- Secondary firewall became active during failover
- Some traffic was not passing correctly
- Connectivity was inconsistent after failover
- Traffic appeared to be forwarded toward the wrong path
- Firewall interface status was up
- Switch interface status was up
- End-to-end traffic was still affected

This created confusion because the firewall HA dashboard/status did not clearly show the actual forwarding issue.

## 5. Initial Assumptions

At first, the issue appeared to be related to firewall HA or firewall policy.

Possible suspected causes included:

- HA configuration mismatch
- Firewall policy issue
- Interface monitoring problem
- Wrong active/standby role
- VLAN or trunk issue
- Routing issue
- ARP issue
- Switch uplink problem
- Session table issue
- NAT issue

However, the firewall HA status was stable and the physical interfaces were also up.

This indicated that the issue might not be purely related to firewall HA configuration.

## 6. Troubleshooting Process

The troubleshooting started from the firewall side and then moved toward the switching layer.

### Firewall HA Verification

The following checks were performed:

- Verified HA primary/secondary status
- Checked active and standby firewall roles
- Confirmed HA heartbeat status
- Checked interface monitoring
- Verified firewall interface status
- Reviewed relevant security policies
- Checked routing behavior
- Checked NAT behavior
- Reviewed traffic/session logs

Result:

```text
Firewall HA status: Healthy
Firewall role change: Working
Firewall interfaces: Up
Firewall policy: No obvious issue found
```

### Connectivity Testing

Connectivity was tested before and after failover.

The issue appeared after traffic shifted toward the standby firewall.

This showed that the failover event was happening, but the network path was not fully adapting to the new active firewall.

### Switching Layer Verification

The connected switch MAC address table was then checked.

A stale or ghost MAC address entry was found.

The switch was still associating the firewall-related MAC address with the old or wrong port instead of the correct active firewall path.

This caused traffic to be forwarded incorrectly after failover.

## 7. Root Cause

The root cause was a ghost or stale MAC address entry on the connected switching layer.

After firewall HA failover, the switch did not properly update the MAC address forwarding entry.

Because of this, traffic was still being forwarded toward the previous or wrong firewall interface path.

The firewall HA mechanism was working, but the Layer 2 forwarding table was not updated correctly.

## 8. Root Cause Summary

```text
Firewall HA status:
✔ Healthy

Firewall failover:
✔ Completed

Physical interfaces:
✔ Up

Actual issue:
✘ Stale/Ghost MAC entry on switch
✘ Traffic forwarded toward wrong port/path
```

The problem was not that the firewall HA failed.

The problem was that the connected switch retained an incorrect MAC forwarding entry after failover.

## 9. Fix Applied

The stale MAC address entry was cleared from the switch.

After clearing the MAC table or refreshing the Layer 2 forwarding state, the switch relearned the correct MAC address from the correct active firewall path.

After the fix:

```text
Firewall HA: Stable
Active firewall path: Correct
Switch MAC table: Correctly relearned
Traffic forwarding: Restored
Failover behavior: Normal
```

## 10. Verification After Fix

After applying the fix, the following checks were performed:

- Verified firewall HA role
- Checked switch MAC address table
- Confirmed the correct port/interface learned the MAC address
- Tested traffic through the active firewall
- Re-tested failover behavior
- Verified user/application connectivity
- Checked logs for repeated failover or interface flaps
- Confirmed that traffic was following the expected path

The traffic started forwarding correctly after the switch relearned the MAC address.

## 11. Technical Explanation

Firewall HA does not only depend on firewall role status.

The connected network must also learn and forward traffic toward the correct active firewall interface.

If a switch keeps an old MAC address entry, traffic may continue going toward the wrong path even when the firewall HA status looks correct.

This is why HA troubleshooting should include both:

- Firewall control plane verification
- Switching layer forwarding verification

A firewall can show active, but traffic can still fail if the switching layer has stale Layer 2 information.

## 12. Useful Checks

During similar issues, the following checks can help on the switching side:

```text
show mac address-table
show mac address-table | include <mac-address>
show interface status
show interface counters
show spanning-tree vlan <vlan-id>
show logging
show arp
```

On the firewall side, verify:

```text
HA status
Interface status
Interface monitoring
Route table
ARP table
Session table
Policy hit count
Traffic logs
NAT logs
```

## 13. Prevention Recommendations

To reduce similar issues in future HA migrations or failover testing:

- Verify MAC address movement during HA failover
- Check switch MAC table before and after failover
- Confirm correct VLAN and trunk forwarding
- Monitor interface flaps and STP changes
- Test failover during a controlled maintenance window
- Include switching layer checks in firewall HA test plans
- Document expected active/standby traffic paths
- Validate traffic flow, not only HA dashboard status
- Keep rollback and verification steps ready before production failover testing

## 14. Key Lessons

- Firewall HA status alone does not guarantee traffic forwarding is correct.
- Layer 2 MAC learning is critical during HA failover.
- A stale MAC entry can make a healthy HA pair look faulty.
- Always verify the switch MAC table during firewall failover issues.
- Interface up does not always mean traffic is taking the correct path.
- Troubleshooting must include firewall, switch, ARP, MAC table, and traffic flow.
- A healthy control plane does not always mean a healthy data path.

## 15. One-Line Takeaway

The firewall HA was working, but traffic failed because the switch had a stale MAC entry pointing traffic toward the wrong firewall path.
