# Routing Case Study: Route Exists but Return Path Missing

> Note: This case study is sanitized for public sharing. Organization names, real IP addresses, hostnames, interface names, screenshots, and sensitive topology details have been removed, changed, or generalized.

## 1. Overview

This case study documents a routing troubleshooting scenario where one side had a valid route toward the destination, but communication still failed because the return path was missing.

At first, the issue looked confusing because the route to the destination network was present. From the local side, the network team could see that traffic had a valid path toward the remote subnet or server.

However, end-to-end connectivity still failed.

After step-by-step troubleshooting, the issue was traced to a missing or incorrect return route on the opposite side.

The key lesson is that routing must work in both directions. A forward route alone is not enough.

## 2. Environment

The environment included:

- Core/distribution routing layer
- Firewall or gateway device
- Internal user subnet
- Remote/server subnet
- Static routes or dynamic routing
- Inter-VLAN or inter-site communication
- Application/server traffic
- Production users

## 3. Expected Traffic Flow

The expected traffic flow was:

```text
Source Network
   ↓
Local Gateway / Router / Firewall
   ↓
Transit Network
   ↓
Remote Gateway / Router / Firewall
   ↓
Destination Server / Network
```

For communication to work, traffic must also return:

```text
Destination Server / Network
   ↓
Remote Gateway / Router / Firewall
   ↓
Transit Network
   ↓
Local Gateway / Router / Firewall
   ↓
Source Network
```

Expected routing condition:

```text
Forward route: Present
Return route: Present
Security policy: Allowed
NAT behavior: Correct
```

## 4. Problem Observed

Users reported that they could not access a remote server or application.

Observed symptoms included:

- Source device had correct IP configuration
- Local gateway was reachable
- Route to destination subnet existed on the local side
- Firewall/security policy appeared to allow traffic
- Destination subnet was known in the route table
- Ping/application access still failed
- Traffic appeared to leave the local side
- No successful response came back

This created confusion because the local routing table looked correct.

## 5. Initial Assumptions

At first, the issue appeared to be related to one of the following:

- Local route missing
- Firewall policy blocking traffic
- Destination server down
- DNS issue
- NAT issue
- Wrong gateway on source device
- Wrong subnet mask
- ISP/WAN issue
- Application service issue
- Remote-side routing issue

Because the local route existed, the investigation moved toward end-to-end path validation.

## 6. Troubleshooting Process

The troubleshooting process followed the packet path from source to destination and then checked the return direction.

### 6.1 Source-Side Verification

The source device was checked first.

Validation included:

- IP address
- Subnet mask
- Default gateway
- DNS settings
- Ping to gateway
- Ping to destination
- Traceroute to destination

Result:

```text
Source IP: Correct
Gateway: Reachable
Local LAN: Working
Destination access: Failed
```

This confirmed that the source device was connected properly to the local network.

### 6.2 Local Route Verification

The local gateway/router/firewall route table was reviewed.

Result:

```text
Route to destination subnet: Present
Next-hop: Correct
Outgoing interface: Correct
```

This showed that the local side knew where to send traffic.

### 6.3 Security Policy Verification

Firewall/security policies were checked.

Validation included:

- Source zone
- Destination zone
- Source subnet
- Destination subnet
- Service/application
- Action allow/deny
- Policy hit count
- Logs

Result:

```text
Policy: Allowed
Policy hit count: Increasing
No obvious local deny found
```

This reduced the possibility of a local firewall block.

### 6.4 Traffic Flow Check

Traffic logs/session tables were reviewed.

Observation:

```text
Traffic was seen leaving the local side toward the destination.
No proper return traffic was observed.
```

This was the key clue.

If traffic leaves but no reply comes back, the issue may be on the destination side, return route, NAT, or remote firewall policy.

### 6.5 Remote-Side Verification

The remote side route table was checked.

This is where the issue was identified.

The remote side did not have a proper route back to the source subnet, or the return route pointed to the wrong gateway/path.

Possible findings:

```text
Remote side had no route back to source subnet
Remote side had default route through another path
Remote firewall did not know how to return traffic
Return traffic was going through a different/asymmetric path
```

## 7. Root Cause

The root cause was a missing or incorrect return route on the remote side.

The local side had a route toward the destination, so traffic was sent correctly in the forward direction.

However, the destination/remote side did not have a valid path back to the source network.

Because of this, replies could not return to the original source.

## 8. Root Cause Summary

```text
Source network:
✔ Working

Local route to destination:
✔ Present

Local firewall policy:
✔ Allowed

Traffic leaving local side:
✔ Yes

Actual issue:
✘ Return route missing or incorrect
✘ Reply traffic could not reach source network
```

The forward path was working.

The return path was missing.

## 9. Fix Applied

The return route was added or corrected on the remote side.

Depending on the environment, the fix may include:

- Adding a static route back to the source subnet
- Advertising the source subnet in dynamic routing
- Correcting next-hop toward the local site
- Updating remote firewall route table
- Correcting asymmetric routing
- Updating remote-side security policy
- Adjusting NAT exemption if required

Example fix:

```text
Remote side route:
Destination: Source subnet
Next-hop: Transit/local site gateway
Interface: Correct return path interface
```

## 10. Verification After Fix

After adding/correcting the return route, the following verification was performed:

- Ping from source to destination
- Ping from destination to source
- Traceroute in both directions
- Firewall session check
- Policy hit count verification
- Route table verification on both sides
- Application access test
- Logs checked for denies/drops

Result:

```text
Forward path: Working
Return path: Working
Ping: Successful
Application access: Restored
Firewall sessions: Established
```

## 11. Technical Explanation

Routing is not only about sending traffic from source to destination.

For communication to work, the reply traffic must also know how to return.

A common mistake is checking only one route table.

For example:

```text
Site A knows how to reach Site B
But Site B does not know how to reach Site A
```

In this case, traffic may leave Site A successfully, but the reply from Site B is dropped, misrouted, or sent to another gateway.

That is why route verification must always be done in both directions.

## 12. Useful Checks

Useful routing checks:

```text
show ip route
show route
show routing-table
show ip route <destination>
traceroute <destination>
ping <destination>
ping source <source-interface>
```

Useful firewall/session checks:

```text
Check session table
Check traffic logs
Check policy hit count
Check NAT logs
Check deny/drop logs
Check source and destination zones
Check return traffic direction
```

Useful validation questions:

```text
Does the source side have a route to the destination?
Does the destination side have a route back to the source?
Is the next-hop correct on both sides?
Is the return traffic using the same expected path?
Is NAT changing the source IP?
Is firewall policy allowing both directions?
Is there asymmetric routing?
Is the destination host using the correct default gateway?
```

## 13. Prevention Recommendations

To avoid similar issues:

- Always verify routing in both directions
- Do not stop after confirming the forward route
- Check return route before blaming the application
- Verify firewall logs for reply traffic
- Document source and destination subnets
- Confirm remote-side gateway and routes
- Use traceroute from both sides when possible
- Review NAT behavior during inter-site communication
- Avoid asymmetric paths unless they are intentionally designed
- Include return-path validation in change checklists

## 14. Key Lessons

- A route on one side does not guarantee full connectivity.
- Communication requires both forward and return paths.
- Traffic can leave successfully and still fail if replies cannot return.
- Missing return routes are a common cause of silent connectivity failure.
- Firewall sessions/logs can show whether return traffic is coming back.
- Always troubleshoot routing from both directions.
- End-to-end testing is stronger than checking only one routing table.

## 15. One-Line Takeaway

The route to the destination existed, but connectivity failed because the remote side did not have a correct return path back to the source network.
