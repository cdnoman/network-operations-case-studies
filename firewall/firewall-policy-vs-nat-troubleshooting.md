# Firewall Case Study: Policy Allowed but Internet Still Not Working Due to NAT Issue

> Note: This case study is sanitized for public sharing. Organization names, real IP addresses, hostnames, interface names, screenshots, and sensitive topology details have been removed, changed, or generalized.

## 1. Overview

This case study documents a firewall troubleshooting scenario where user traffic was allowed by the security policy, but internet access was still not working.

At first, the issue looked like a firewall policy problem. The users were matching the correct policy, and traffic was allowed, but browsing and external connectivity were failing.

After step-by-step troubleshooting, the issue was traced to NAT. The firewall policy was allowing the traffic, but the source NAT rule was missing, incorrect, or not matching the required source zone/subnet.

The key lesson is that firewall policy and NAT are two different checks. A policy can allow traffic, but without correct NAT, private IP traffic may still fail on the internet.

## 2. Environment

The environment included:

- Next-generation firewall
- Internal users zone
- WAN/internet zone
- Security policy
- Source NAT policy
- Default route toward ISP
- Internal private IP subnets
- Internet access requirement
- Production users

## 3. Expected Traffic Flow

The expected traffic flow was:

```text
User PC
   ↓
Internal Switch/Gateway
   ↓
Firewall Inside Interface
   ↓
Security Policy Match
   ↓
Source NAT Applied
   ↓
WAN Interface
   ↓
Internet
```

For internet access to work, both policy and NAT must be correct.

Expected behavior:

```text
Private user IP
   ↓
Firewall policy allows traffic
   ↓
SNAT translates private IP to public/WAN IP
   ↓
Traffic goes to internet
   ↓
Return traffic comes back to firewall
   ↓
Firewall maps return traffic back to internal user
```

## 4. Problem Observed

Users reported that internet access was not working.

Observed symptoms included:

- User IP configuration was correct
- Gateway was reachable
- Firewall policy existed
- Policy hit count was increasing
- Traffic appeared to be allowed
- DNS or browsing was failing
- External ping or web access was not working
- No clear deny log was visible for the user traffic

This created confusion because the firewall policy looked correct.

## 5. Initial Assumptions

At first, the issue appeared to be related to one of the following:

- Security policy missing
- Wrong source zone
- Wrong destination zone
- ISP issue
- Default route issue
- DNS issue
- Firewall deny rule
- User endpoint issue
- WAN interface issue
- NAT issue

Because the policy was matching and allowing traffic, the investigation moved beyond only firewall policy.

## 6. Troubleshooting Process

The troubleshooting process followed the traffic path from user to internet.

### 6.1 User-Side Verification

The affected user device was checked first.

Validation included:

- IP address
- Subnet mask
- Default gateway
- DNS settings
- Ping to gateway
- Ping to firewall inside interface
- Basic LAN connectivity

Result:

```text
User IP address: Correct
Default gateway: Reachable
LAN connectivity: Working
Internet access: Failed
```

This confirmed that the user was connected to the internal network.

### 6.2 Firewall Policy Verification

The firewall security policy was reviewed.

Checked items included:

- Source zone
- Destination zone
- Source address object
- Destination address
- Service/application
- Schedule
- Action allow/deny
- Policy hit count
- Logs

Result:

```text
Security policy: Matched
Action: Allowed
Policy hit count: Increasing
Deny logs: No obvious deny found
```

This showed that the firewall was not simply blocking the traffic by policy.

### 6.3 Routing Verification

The firewall route table was checked.

Validation included:

- Default route
- WAN next-hop
- Interface status
- Route priority/distance
- Return path possibility

Result:

```text
WAN interface: Up
Default route: Present
Next-hop: Reachable
Routing: No obvious issue found
```

Routing did not appear to be the main issue.

### 6.4 NAT Verification

The NAT policy was then reviewed.

This is where the issue was identified.

The user subnet/source zone was not matching the expected SNAT rule, or the NAT rule was missing/incorrect for that traffic.

Possible NAT issues included:

- Source zone missing from SNAT rule
- Source subnet/address object not included
- Wrong outgoing interface selected
- NAT rule order issue
- NAT disabled on the matching policy
- Incorrect translated address/interface IP
- Overlapping NAT rule matching before correct rule

Result:

```text
Security policy: Allowed traffic
SNAT policy: Missing or not matching correctly
Internet access: Failed
```

## 7. Root Cause

The root cause was incorrect or missing source NAT for the affected user traffic.

The firewall security policy was allowing the traffic, but the private source IP was not being translated properly before going to the internet.

Because private IP addresses are not routable on the public internet, the return traffic could not come back correctly.

## 8. Root Cause Summary

```text
User LAN connectivity:
✔ Working

Firewall security policy:
✔ Allowed

WAN/default route:
✔ Present

Actual issue:
✘ SNAT missing or not matching
✘ Private IP not translated correctly
✘ Return traffic failed
```

The firewall policy was not the problem.

The NAT policy was the missing part.

## 9. Fix Applied

The SNAT policy was corrected.

Depending on the environment, the fix may include:

- Adding the correct source zone
- Adding the correct user subnet/address object
- Selecting the correct WAN/outgoing interface
- Enabling NAT on the internet access policy
- Correcting NAT rule order
- Using the correct translated IP/interface IP
- Removing conflicting/overlapping NAT rules

Example fix:

```text
Source zone: Users
Source subnet: Affected user subnet
Destination zone: WAN
Destination: Any/Internet
Translation: Outgoing interface IP / assigned public IP
Action: SNAT enabled
```

## 10. Verification After Fix

After correcting the NAT configuration, the following checks were performed:

- User internet browsing test
- Ping to public IP
- DNS resolution test
- Firewall policy hit count
- NAT translation/session table
- Traffic logs
- Return traffic confirmation
- Multiple user testing

Result:

```text
Internet access: Restored
SNAT translation: Working
Firewall sessions: Established
Return traffic: Working
User browsing: Successful
```

## 11. Technical Explanation

Firewall policy and NAT are related, but they are not the same.

A security policy decides whether traffic is allowed or denied.

NAT decides whether the source or destination IP address should be translated.

For internet access, internal private IP addresses usually need source NAT before they go out to the public internet.

That means:

```text
Policy allow = traffic is permitted
SNAT = private IP is translated for internet routing
```

If the policy allows traffic but SNAT is missing, the packet may leave with a private source IP or fail to match the correct outbound translation.

The result is that internet access fails even though the firewall policy looks correct.

## 12. Useful Checks

Useful firewall checks:

```text
Check policy hit count
Check traffic logs
Check NAT logs
Check session table
Check route table
Check source zone
Check destination zone
Check source address object
Check outgoing interface
Check NAT rule order
Check translated source IP
```

Useful network checks:

```text
ping <gateway>
ping 8.8.8.8
nslookup google.com
traceroute 8.8.8.8
test from affected user subnet
test from another working subnet
```

Useful validation questions:

```text
Is the firewall policy matching?
Is the policy allowing traffic?
Is NAT enabled or applied?
Is the correct source zone included?
Is the correct source subnet included?
Is the correct WAN interface selected?
Is another NAT rule matching before this rule?
Is return traffic coming back?
```

## 13. Prevention Recommendations

To avoid similar issues:

- Always verify policy and NAT together
- Keep clear naming for NAT rules
- Match NAT rules with source zones and subnets
- Review NAT rule order after every change
- Test from every required user zone
- Document which subnets use which WAN/NAT path
- Do not assume policy hit count means internet will work
- Include NAT verification in firewall change checklists
- Compare working and non-working subnet behavior
- Confirm sessions and translations after applying changes

## 14. Key Lessons

- Firewall policy allowed does not always mean internet will work.
- NAT is required for most private-to-internet traffic.
- A policy hit count only proves the policy is matching.
- It does not prove that translation is correct.
- Always check source zone, source subnet, NAT rule, route, and return path.
- Many internet issues are not deny-policy issues; they are NAT matching issues.
- Troubleshooting should follow the full packet path, not only the policy table.

## 15. One-Line Takeaway

The firewall was allowing the traffic, but internet failed because the user subnet was missing from the correct SNAT rule.
