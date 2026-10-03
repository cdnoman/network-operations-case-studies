# Switching Case Study: VLAN Mismatch Causing Server Reachability Issue

> Note: This case study is sanitized for public sharing. Organization names, real IP addresses, hostnames, VLAN IDs, interface names, screenshots, and sensitive topology details have been removed, changed, or generalized.

## 1. Overview

This case study documents a switching troubleshooting scenario where users were unable to reach an internal server after a network change.

At first, the issue looked like a server-side or firewall problem because the server was powered on, its interface was up, and other services appeared normal. However, after step-by-step troubleshooting, the issue was traced to a VLAN mismatch on the switching path.

The affected user network and the server network were not being carried correctly across the switch path due to an incorrect VLAN assignment/trunk configuration.

## 2. Environment

The environment included:

- Access switches
- Distribution/core switching layer
- User VLAN
- Server VLAN
- Trunk links
- Access ports
- Inter-VLAN routing
- Firewall or gateway device
- Internal application/server
- Production users

## 3. Expected Traffic Flow

In the expected design, user traffic should follow this path:

```text
User PC
   ↓
Access Switch
   ↓
Distribution/Core Switch
   ↓
Gateway / Firewall / L3 SVI
   ↓
Server VLAN
   ↓
Internal Server
```

The user VLAN and server VLAN must be correctly configured and allowed across the required switch path.

## 4. Problem Observed

Users reported that they could not reach an internal server/application.

Observed symptoms included:

- User PCs were connected to the network
- User switch ports were up
- Server interface was up
- Server was reachable from some parts of the network
- Affected users could not access the server/application
- Ping or application access failed from the affected user segment
- No obvious physical link failure was visible

This created confusion because the issue did not look like a simple cable or interface-down problem.

## 5. Initial Assumptions

At first, the issue appeared to be related to one of the following:

- Server service issue
- Server firewall issue
- Network firewall policy issue
- Gateway issue
- Routing issue
- DNS issue
- User endpoint issue
- Access switch issue
- Inter-VLAN communication issue

However, the server was reachable from other locations, which indicated that the server itself was most likely not fully down.

## 6. Troubleshooting Process

The troubleshooting started from the user side and then moved toward the switching path.

### 6.1 User Side Testing

The affected user PC was checked first.

Validation included:

- IP address
- Default gateway
- DNS settings
- VLAN assignment
- Local connectivity
- Ping to gateway
- Ping to server
- Application access test

Result:

```text
User IP address: Correct
Default gateway: Reachable
Local network: Working
Server/application access: Failed
```

This showed that the user device was connected to the network, but traffic toward the server was failing.

### 6.2 Server Side Testing

The server was then checked from other network segments.

Result:

```text
Server power/status: Up
Server interface: Up
Server reachable from other segments: Yes
Server unreachable from affected VLAN/users: Yes
```

This confirmed that the server was not completely down.

The problem was related to a specific network path.

### 6.3 Gateway and Routing Verification

Routing and gateway checks were performed.

The gateway/SVI for the user VLAN and server VLAN was reviewed.

Result:

```text
User VLAN gateway: Working
Server VLAN gateway: Working
Routing table: No obvious missing route
```

This reduced the possibility of a pure routing issue.

### 6.4 Switching Path Verification

The switching path between the affected users and the server network was checked.

The following areas were reviewed:

- Access port VLAN assignment
- Trunk port allowed VLANs
- VLAN existence on switches
- STP state
- MAC address learning
- Interface status
- Interface counters

During this review, a VLAN mismatch was found.

The required VLAN was either not assigned correctly on an access port or not allowed correctly on a trunk path.

## 7. Root Cause

The root cause was incorrect VLAN configuration in the switching path.

The affected traffic was not being forwarded correctly because the required VLAN was missing/mismatched on the switch path.

This caused traffic from the user segment to fail when trying to reach the server/application.

## 8. Root Cause Summary

```text
Physical link status:
✔ Up

User endpoint:
✔ Connected

Server:
✔ Running

Gateway/routing:
✔ Appeared normal

Actual issue:
✘ VLAN mismatch / VLAN not allowed correctly
✘ Traffic not forwarding through expected Layer 2 path
```

The issue was not the server.

The issue was not the physical link.

The issue was a VLAN forwarding problem in the switching path.

## 9. Fix Applied

The VLAN configuration was corrected on the switching path.

Depending on the exact mismatch, the fix may include:

- Correcting the access port VLAN
- Allowing the required VLAN on the trunk
- Creating the missing VLAN on the switch
- Correcting native VLAN mismatch
- Confirming the correct uplink/trunk path
- Verifying STP state for the VLAN

Example correction:

```text
Required VLAN added/allowed on trunk path
Access port assigned to correct VLAN
Switching path verified end-to-end
```

## 10. Verification After Fix

After the VLAN configuration was corrected, the following verification was performed:

- User gateway reachable
- Server reachable from affected users
- Application access restored
- MAC address learning confirmed
- VLAN allowed on trunk confirmed
- STP state checked
- Interface counters reviewed
- No packet drops/errors observed

Result:

```text
Affected users: Able to reach server
Application access: Restored
Server reachability: Working
VLAN forwarding: Correct
```

## 11. Technical Explanation

A VLAN issue can look like many other problems.

It may look like:

- Server down
- Firewall blocking
- DNS issue
- Gateway issue
- Routing issue
- User PC issue

But if the VLAN is not carried correctly across the switch path, traffic will not reach the expected destination even when interfaces are up.

This is why interface status alone is not enough.

A port can be up, but traffic can still fail if:

- The access VLAN is wrong
- The VLAN is missing on the switch
- The VLAN is not allowed on the trunk
- STP is blocking the VLAN path
- Native VLAN mismatch exists
- MAC learning is happening on the wrong path

## 12. Useful Checks

During similar issues, the following checks are useful:

```text
show vlan brief
show interfaces trunk
show interfaces status
show interfaces switchport
show mac address-table
show mac address-table vlan <vlan-id>
show spanning-tree vlan <vlan-id>
show interface counters
show ip interface brief
show ip route
ping <gateway>
ping <server-ip>
traceroute <server-ip>
```

Useful validation questions:

```text
Is the user port in the correct VLAN?
Does the VLAN exist on all required switches?
Is the VLAN allowed on trunk links?
Is STP forwarding for this VLAN?
Is the gateway reachable from the user VLAN?
Is the server reachable from another VLAN?
Is the MAC address learned on the expected interface?
```

## 13. Prevention Recommendations

To avoid similar issues in the future:

- Document VLAN-to-port mapping
- Verify trunk allowed VLANs after every change
- Use standard switchport templates
- Keep VLAN naming consistent across switches
- Check VLAN existence before assigning ports
- Validate STP state after changes
- Test from affected and unaffected VLANs
- Keep rollback configuration ready
- Include Layer 2 verification in change plans
- Do not assume “interface up” means “traffic path working”

## 14. Key Lessons

- Interface up does not always mean traffic is forwarding correctly.
- A VLAN mismatch can look like a server, firewall, or routing problem.
- Always verify access VLAN and trunk allowed VLANs.
- Check MAC learning to confirm where traffic is actually going.
- Troubleshooting should follow the path from user to server.
- Layer 2 issues can break application access even when Layer 3 looks normal.
- Small VLAN mistakes can create big production impact.

## 15. One-Line Takeaway

The server was not down. The traffic was failing because the required VLAN was mismatched or not carried correctly across the switching path.
