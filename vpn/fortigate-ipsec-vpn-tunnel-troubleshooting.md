# IPsec VPN Case Study: Tunnel Up but Traffic Not Passing

> Note: This case study is sanitized for public sharing. Organization names, real IP addresses, peer IPs, internal subnets, hostnames, screenshots, and sensitive topology details have been removed, changed, or generalized.

## 1. Overview

This case study documents an IPsec VPN troubleshooting scenario where the VPN tunnel appeared to be up, but traffic was not passing between the local and remote networks.

At first, the tunnel status looked healthy. Phase 1 was up, Phase 2 was active, and the VPN dashboard showed the tunnel as established. However, users were still unable to access the remote-side resources.

After step-by-step troubleshooting, the issue was traced to a traffic-path problem rather than the VPN tunnel establishment itself.

The key lesson is that an IPsec tunnel being up does not always mean end-to-end traffic is working.

## 2. Environment

The environment included:

- FortiGate firewall
- Remote VPN peer firewall
- Site-to-site IPsec VPN
- Local internal subnet
- Remote internal subnet
- Phase 1 configuration
- Phase 2 selectors
- Static route or policy route
- Firewall policies
- NAT exemption
- Internal users/servers
- Production application traffic

## 3. Expected Traffic Flow

The expected traffic flow was:

```text
Local User / Server
   ↓
Local Firewall
   ↓
IPsec VPN Tunnel
   ↓
Remote Firewall
   ↓
Remote Server / Application
```

For VPN traffic to work properly, all required components must match:

- Phase 1 must be established
- Phase 2 must match the correct local and remote subnets
- Firewall policy must allow LAN-to-VPN traffic
- Firewall policy must allow VPN-to-LAN return traffic
- Route to remote subnet must point to the VPN tunnel
- NAT should not incorrectly translate VPN traffic
- Remote side must have return route/policy
- Interesting traffic must match the VPN selectors

## 4. Problem Observed

Users reported that they could not access resources on the remote side.

Observed symptoms included:

- VPN tunnel status showed up
- Phase 1 appeared established
- Phase 2 appeared active
- Local users could reach local gateway
- Remote application/server was unreachable
- Ping or application traffic failed across VPN
- No physical WAN issue was visible
- Internet/WAN link was working normally

This created confusion because the VPN looked up from the dashboard.

## 5. Initial Assumptions

At first, the issue appeared to be related to one of the following:

- Remote server down
- VPN tunnel issue
- Phase 2 mismatch
- Firewall policy missing
- Static route missing
- NAT issue
- Remote-side return route missing
- Encryption domain mismatch
- Asymmetric routing
- Interesting traffic not matching the VPN selector

Because the tunnel was showing up, the troubleshooting moved toward traffic flow validation.

## 6. Troubleshooting Process

The troubleshooting process started by verifying tunnel status and then checking traffic flow from source to destination.

### 6.1 VPN Status Verification

The VPN status was checked first.

Validation included:

- Phase 1 status
- Phase 2 status
- Peer connectivity
- Tunnel uptime
- Encryption/authentication status
- VPN logs

Result:

```text
Phase 1: Up
Phase 2: Up
VPN tunnel status: Established
WAN connectivity: Working
```

This confirmed that the VPN negotiation itself was successful.

However, this did not prove that user traffic was passing.

### 6.2 Local User Connectivity Check

The local user/source device was checked.

Validation included:

- IP address
- Default gateway
- Local LAN connectivity
- Ping to local gateway
- Ping to remote server
- Application access test

Result:

```text
Local user IP: Correct
Local gateway: Reachable
Local LAN: Working
Remote subnet access: Failed
```

This showed that the issue was beyond the local LAN.

### 6.3 Route Verification

The firewall route table was checked.

Validation included:

- Route to remote subnet
- Tunnel interface route
- Static route/policy route
- Route priority/distance
- Matching destination subnet

Possible issue:

```text
VPN tunnel is up, but route to remote subnet is missing or pointing to the wrong interface.
```

A VPN tunnel can be established, but if the firewall does not route traffic into the tunnel, user traffic will never reach the remote side.

### 6.4 Firewall Policy Verification

Firewall policies were reviewed.

Checked items included:

- LAN-to-VPN policy
- VPN-to-LAN return policy
- Source subnet
- Destination subnet
- Service/application
- Schedule
- Action allow/deny
- Policy hit count
- Logs

Possible issue:

```text
Tunnel is up, but firewall policy is not allowing the required traffic.
```

If the policy is missing or not matching, the tunnel may remain up, but traffic will be denied or dropped.

### 6.5 Phase 2 Selector Verification

Phase 2 selectors/encryption domains were checked.

Validated items included:

- Local subnet
- Remote subnet
- Subnet mask
- Address object
- Proxy ID / selector match
- Multiple subnet requirements
- Remote-side matching configuration

Possible issue:

```text
Phase 2 is up for one subnet pair, but user traffic is using a subnet not included in the selector.
```

If user traffic does not match the Phase 2 selector, it may not enter the tunnel correctly.

### 6.6 NAT Verification

NAT behavior was reviewed.

For site-to-site VPN traffic, NAT is usually not required between private site subnets unless the design specifically requires it.

Possible issue:

```text
VPN traffic is being source NATed like internet traffic.
```

If VPN traffic is incorrectly NATed, the remote side may receive unexpected source IPs and may not route traffic back correctly.

### 6.7 Remote-Side Verification

The remote side also needed verification.

Possible remote-side issues included:

- Remote firewall policy missing
- Remote return route missing
- Remote Phase 2 selector mismatch
- Remote server firewall blocking traffic
- Remote host using wrong default gateway
- Remote NAT issue
- Remote security policy not allowing return traffic

This step is important because a tunnel can look healthy from the local firewall, but the failure may exist on the remote firewall or remote server path.

## 7. Root Cause

The root cause in this type of issue is usually not Phase 1 establishment.

The tunnel can be up, but traffic may still fail due to one of the following:

- Route to remote subnet missing or wrong
- Firewall policy not matching
- Phase 2 selector mismatch
- NAT exemption missing
- Return route missing on remote side
- Remote-side policy missing
- Wrong source/destination subnet in address objects
- Asymmetric traffic path

In this case study, the tunnel was established, but the traffic path was incomplete.

The VPN was up at the control-plane level, but data-plane forwarding was not working correctly.

## 8. Root Cause Summary

```text
VPN Phase 1:
✔ Up

VPN Phase 2:
✔ Up / Appeared active

WAN connectivity:
✔ Working

Actual issue:
✘ Traffic path incomplete
✘ Route / policy / NAT / selector / return path issue
✘ User traffic not passing end-to-end
```

The VPN tunnel status was not the final proof.

End-to-end traffic verification was required.

## 9. Fix Applied

The fix depends on the exact root cause found during troubleshooting.

Possible fixes include:

### If route was missing

```text
Add route to remote subnet via VPN tunnel interface
```

### If firewall policy was missing

```text
Create/modify LAN-to-VPN and VPN-to-LAN policies
```

### If Phase 2 selector was wrong

```text
Correct local and remote subnet selectors on both sides
```

### If NAT was affecting VPN traffic

```text
Add NAT exemption or disable NAT for VPN-bound traffic
```

### If return route was missing

```text
Add route on remote side back to local subnet
```

### If remote server was blocking traffic

```text
Allow required traffic on remote server firewall/application
```

After correcting the traffic-path issue, traffic successfully passed through the VPN tunnel.

## 10. Verification After Fix

After applying the fix, the following verification was performed:

- VPN tunnel status checked
- Phase 1 status checked
- Phase 2 status checked
- Route table verified
- Firewall policy hit count checked
- NAT behavior verified
- Ping tested between local and remote hosts
- Application access tested
- VPN traffic counters checked
- Logs reviewed for denies/drops
- Return traffic confirmed

Result:

```text
VPN tunnel: Up
Traffic counters: Increasing
Local-to-remote ping: Working
Application access: Restored
Firewall policy: Matching
Return traffic: Working
```

## 11. Technical Explanation

IPsec VPN has two important parts:

```text
Control plane = Tunnel negotiation
Data plane = Actual user traffic forwarding
```

Phase 1 and Phase 2 being up means the firewalls were able to negotiate the VPN tunnel.

But user traffic still needs:

- Correct route
- Correct policy
- Correct Phase 2 selector
- Correct NAT behavior
- Correct return path
- Remote-side permission

That is why a green VPN status does not always mean the application will work.

A tunnel can be up, but traffic can still fail if packets do not match the correct route, selector, policy, or return path.

## 12. Useful Checks

Useful FortiGate checks:

```text
get vpn ipsec tunnel summary
diagnose vpn tunnel list
diagnose vpn ike gateway list
get router info routing-table all
diagnose firewall iprope lookup
diagnose sys session filter
diagnose sys session list
diagnose debug flow
diagnose sniffer packet any "host <source-ip> or host <destination-ip>" 4
```

Useful traffic checks:

```text
ping <remote-ip>
traceroute <remote-ip>
Check policy hit count
Check VPN tunnel counters
Check route to remote subnet
Check NAT exemption
Check local/remote subnet objects
Check Phase 2 selectors
Check remote-side return route
```

Useful validation questions:

```text
Is Phase 1 up?
Is Phase 2 up?
Is user traffic matching the Phase 2 selector?
Is there a route to the remote subnet?
Is LAN-to-VPN policy allowing the traffic?
Is VPN-to-LAN return policy allowing the return traffic?
Is NAT disabled/exempted for VPN traffic?
Does the remote side have a route back?
Is the remote server allowing the traffic?
Are VPN counters increasing?
```

## 13. Prevention Recommendations

To avoid similar issues:

- Document local and remote encryption domains
- Confirm Phase 2 selectors with the remote team
- Keep firewall address objects clear and accurate
- Add VPN routes before testing users
- Verify NAT exemption for VPN traffic
- Test both directions after tunnel creation
- Check policy hit count and tunnel counters
- Confirm remote-side return routes
- Keep a VPN troubleshooting checklist
- Do not rely only on green tunnel status

## 14. Key Lessons

- VPN up does not always mean traffic is passing.
- Phase 1/Phase 2 status only confirms tunnel negotiation.
- User traffic still needs correct routing, policy, NAT, and return path.
- Phase 2 selector mismatch is a common cause of silent VPN traffic failure.
- NAT can break VPN traffic if exemption is missing.
- Remote-side firewall and routing must also be verified.
- Always confirm traffic counters, not only tunnel status.

## 15. One-Line Takeaway

The VPN tunnel was up, but traffic was not passing because the end-to-end route, policy, NAT, selector, or return path was incomplete.
