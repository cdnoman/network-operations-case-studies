# Switching Case Study: STP Blocking vs Port Down Misdiagnosis

> Note: This case study is sanitized for public sharing. Organization names, real IP addresses, hostnames, interface names, VLAN IDs, screenshots, and sensitive topology details have been removed, changed, or generalized.

## 1. Overview

This case study documents a switching troubleshooting scenario where a network link was initially suspected to be down, but the actual issue was related to Spanning Tree Protocol behavior.

In the environment, redundant switch paths were available. One interface was physically up, but traffic was not forwarding through it. At first, the port was treated like a failed or down link. However, after proper verification, it was found that the port was not down. It was in an STP blocking/discarding state to prevent a Layer 2 loop.

The key lesson is that a port can be physically up but still not forward traffic.

## 2. Environment

The environment included:

- Access switches
- Distribution/core switches
- Redundant uplinks
- Trunk links
- Multiple VLANs
- STP/RSTP/MSTP environment
- User VLANs
- Server/application traffic
- Production switching path

## 3. Expected Traffic Flow

In a redundant switching design, multiple physical paths may exist between switches.

Expected design:

```text
Access Switch
   ↓
Primary Uplink
   ↓
Distribution/Core Switch
```

Backup path:

```text
Access Switch
   ↓
Secondary Uplink
   ↓
Distribution/Core Switch
```

STP prevents loops by allowing one path to forward and placing the redundant path in a blocking or discarding state.

Expected STP behavior:

```text
Primary path: Forwarding
Backup path: Blocking/Discarding
```

This is normal behavior when redundancy exists.

## 4. Problem Observed

During troubleshooting, one switch uplink was reported as not passing traffic.

Observed symptoms included:

- Physical interface showed up
- Link light was present
- Trunk configuration looked correct
- VLANs appeared configured
- No physical cable issue was visible
- Traffic was not forwarding through one path
- MAC addresses were not learned on the expected interface
- Users or services were affected during path testing

At first, the issue looked like a port-down, trunk, or cable problem.

## 5. Initial Assumptions

The initial assumptions included:

- Cable issue
- SFP/transceiver issue
- Switch port down
- Trunk misconfiguration
- VLAN not allowed
- Duplex/speed issue
- Uplink failure
- Switch hardware issue
- Firewall/gateway issue
- Routing issue

However, the physical port was up and the link was stable.

This indicated that the issue was not simply a Layer 1 failure.

## 6. Troubleshooting Process

The troubleshooting started by checking the physical and logical state of the interface.

### 6.1 Interface Status Check

The interface was checked first.

Result:

```text
Interface physical status: Up
Line protocol: Up
Cable/SFP: Working
Port administratively enabled: Yes
```

This confirmed that the interface was not physically down.

### 6.2 Trunk and VLAN Check

The trunk configuration was reviewed.

Checks included:

- Trunk mode
- Allowed VLANs
- Native VLAN
- VLAN existence
- Interface configuration
- Neighbor switch configuration

Result:

```text
Trunk: Present
Required VLANs: Configured
Native VLAN: No obvious mismatch found
```

This reduced the chance of a basic trunk issue.

### 6.3 MAC Address Learning Check

The MAC address table was checked.

Observation:

```text
MAC addresses were not being learned on the suspected interface.
```

This showed that although the port was physically up, it was not forwarding traffic for the affected VLAN/path.

### 6.4 STP Verification

The STP state for the affected VLAN was checked.

This was the key step.

Result:

```text
Interface state: Blocking/Discarding
Reason: STP loop prevention
```

The port was not down.

It was blocked by STP because another path was already forwarding.

## 7. Root Cause

The root cause was a misunderstanding of the port state.

The interface was physically up, but STP had placed it in a blocking/discarding state to prevent a Layer 2 loop.

The port was not faulty.

The switch was intentionally preventing traffic forwarding on that path.

## 8. Root Cause Summary

```text
Physical link:
✔ Up

Trunk:
✔ Configured

VLAN:
✔ Present

Interface:
✔ Not physically down

Actual issue:
✘ STP placed the port in blocking/discarding state
✘ Port was not forwarding traffic
```

The issue was not a cable failure.

The issue was STP behavior.

## 9. Fix Applied

In this type of case, the fix depends on the design.

If STP blocking is expected, no fix is required. The port is working as designed.

If traffic should forward through a different path, then the STP design must be adjusted carefully.

Possible corrective actions include:

- Verify root bridge placement
- Adjust STP priority if required
- Review primary and backup uplink design
- Confirm VLAN-specific STP state
- Check for unintended Layer 2 loops
- Verify trunk allowed VLANs
- Review port-channel design if multiple links should be active
- Use EtherChannel/LACP if parallel active links are required
- Document forwarding and blocking paths

In this case, the main fix was to correctly identify the state and avoid treating the STP-blocked link as a failed link.

## 10. Verification After Fix

After identifying the actual STP state, the following checks were performed:

- Verified physical link status
- Verified trunk status
- Checked STP state per VLAN
- Confirmed forwarding port
- Confirmed blocking/discarding port
- Checked root bridge
- Checked MAC address learning
- Tested traffic through the active forwarding path
- Confirmed failover behavior when primary path was affected

Result:

```text
Primary path: Forwarding
Redundant path: Blocking/Discarding
Traffic path: Working as per STP design
Interface status: Up
Issue classification: STP behavior, not port down
```

## 11. Technical Explanation

A switch port can be physically up but not forward traffic.

This is common in STP-based Layer 2 redundancy.

STP is designed to prevent switching loops. When multiple Layer 2 paths exist, STP allows one path to forward and blocks another path.

This prevents:

- Broadcast storms
- MAC address flapping
- Duplicate frames
- Layer 2 loops
- Network instability

That is why “interface up” and “traffic forwarding” are two different things.

A port can be:

```text
Physical state: Up
Protocol state: Up
STP state: Blocking/Discarding
Traffic forwarding: No
```

This is normal when STP is protecting the network.

## 12. Useful Checks

Useful switching checks:

```text
show interfaces status
show interfaces trunk
show spanning-tree
show spanning-tree vlan <vlan-id>
show spanning-tree interface <interface-id>
show mac address-table
show mac address-table interface <interface-id>
show logging
show etherchannel summary
show interfaces counters
```

Useful validation questions:

```text
Is the interface physically up?
Is the port forwarding or blocking?
Which switch is the STP root bridge?
Is the STP state same for all VLANs?
Is the VLAN allowed on the trunk?
Is this a redundant path?
Should this link be active or backup?
Is EtherChannel required instead of separate links?
Is MAC learning happening on the expected port?
```

## 13. Prevention Recommendations

To avoid similar confusion:

- Always check STP state before declaring a port down
- Document primary and backup uplinks
- Define the intended root bridge
- Use clear switchport descriptions
- Monitor STP topology changes
- Use EtherChannel/LACP where multiple links should forward together
- Avoid unmanaged redundant Layer 2 paths
- Validate VLAN-specific STP state
- Keep a standard Layer 2 troubleshooting checklist
- Train NOC teams on the difference between link state and forwarding state

## 14. Key Lessons

- Port up does not always mean traffic is forwarding.
- STP blocking is not the same as port down.
- A blocked STP port may be working exactly as designed.
- Redundant Layer 2 paths need proper STP design.
- MAC learning helps confirm whether a port is forwarding traffic.
- Always check STP state per VLAN.
- Do not troubleshoot Layer 2 redundancy only from interface status.

## 15. One-Line Takeaway

The port was not down. STP was blocking the redundant path to prevent a Layer 2 loop.
