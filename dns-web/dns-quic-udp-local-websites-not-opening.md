# DNS/Web Access Case Study: Local Websites Not Opening Due to Split-DNS Design and QUIC/UDP Filtering

> Note: This case study is sanitized for public sharing. Organization names, real domain names, public IPs, internal IPs, hostnames, firewall policy names, screenshots, and sensitive topology details have been removed, changed, or generalized.

## 1. Overview

This case study documents a complex production troubleshooting scenario where multiple internal/local websites and portals were not opening properly for many users.

The issue was not consistent for every user. Some users were able to open the portals, while others could not. In some cases, changing the browser helped temporarily. In other cases, changing DNS settings helped temporarily. Sometimes users were shifted from Chrome to Firefox. Sometimes DNS entries were changed manually. Sometimes public DNS such as `8.8.8.8` was used for testing.

Because the symptoms were inconsistent, the issue was initially difficult to isolate.

After long-term troubleshooting, the issue was found to be related to multiple layers:

- Internal DNS design issue
- Active Directory DNS zone conflict with public website domain
- Browser behavior difference
- QUIC protocol behavior in Chrome
- UDP-based traffic filtering on the internet access gateway
- Required web protocols not fully allowed for some traffic paths

The final resolution required understanding both DNS design and browser/protocol behavior.

## 2. Environment

The environment included:

- Active Directory domain
- Internal DNS servers
- Public-facing websites
- Internal/local portals
- User endpoints
- Chrome browser
- Firefox browser
- Internet access gateway
- Firewall/security filtering
- Local DNS resolution
- Public DNS testing
- Production users

## 3. Problem Statement

Many users were unable to open local websites and internal portals reliably.

The issue was confusing because it did not behave like a simple “website down” problem.

Observed behavior included:

- Website opened for some users but failed for others
- Some portals worked in Firefox but not in Chrome
- Some portals worked after changing DNS
- Some portals worked temporarily after DNS up/down changes
- Some users were tested with public DNS such as `8.8.8.8`
- Local website access was inconsistent
- The issue continued for a long time and affected user productivity

The issue looked like DNS at first, but DNS was not the only problem.

## 4. Original DNS Design Issue

During investigation, one important design issue was identified.

The Active Directory domain was created using the same root domain name as the public website domain.

Example sanitized design:

```text
Public website domain:
example.com

Active Directory internal domain:
example.com
```

A better design would have been:

```text
Active Directory internal domain:
ad.example.com
```

or:

```text
corp.example.com
internal.example.com
```

Because the internal AD DNS zone used the same root domain as the public website domain, internal DNS servers treated that domain as an internal authoritative zone.

This caused problems when internal users tried to access public websites or portals under the same domain name.

## 5. Expected DNS Behavior

For public websites, internal users should be able to resolve the website name properly.

Expected flow:

```text
User Browser
   ↓
Internal DNS Server
   ↓
Correct DNS Resolution
   ↓
Website IP Address Returned
   ↓
Browser Connects to Website
```

If the website is public, the DNS path must return the correct public or internally mapped record.

If split-DNS is used, the required internal DNS records must exist.

## 6. Actual DNS Behavior

Because the internal DNS zone had the same domain name as the public website domain, internal DNS tried to resolve the website names locally.

Actual behavior:

```text
User Browser
   ↓
Internal DNS Server
   ↓
Internal DNS zone checks example.com
   ↓
Required website record missing or incorrect
   ↓
Website resolution fails or returns wrong result
```

This created local website access issues.

From the user side, it looked like:

```text
Website not opening
Portal not loading
Browser error
DNS change sometimes fixes it
Different browser sometimes works
```

But the underlying issue was deeper.

## 7. Browser-Level Complexity

During troubleshooting, another layer was discovered.

Some behavior was different between browsers.

For example:

- Chrome behaved differently from Firefox
- Some portals failed in Chrome but worked in another browser
- Disabling specific Chrome behavior improved access for some portals

The investigation eventually reached Chrome QUIC protocol behavior.

QUIC is commonly associated with UDP-based web transport behavior. When QUIC is enabled, Chrome may try to use UDP-based communication for supported web services.

If the firewall, proxy, secure internet gateway, or IAG blocks or filters the required UDP traffic, some websites or portals may behave inconsistently.

## 8. QUIC and UDP Filtering Issue

The internet access gateway/security device had UDP-related restrictions.

Because QUIC uses UDP-based behavior, blocking or filtering UDP affected some browser traffic.

This created a second issue:

```text
DNS may resolve correctly,
but browser traffic may still fail because required UDP/QUIC behavior is blocked.
```

In some cases, disabling QUIC in Chrome helped certain portals open.

This was a strong clue that the issue was not only DNS.

It was also related to browser protocol behavior and gateway filtering.

## 9. Problem Observed in Simple Form

The issue had multiple symptoms:

```text
Some users could open portals
Some users could not open portals

Some websites worked in Firefox
Some websites failed in Chrome

Changing DNS sometimes helped
Using public DNS sometimes changed behavior

Disabling QUIC helped in some cases

Allowing required protocols on the gateway improved access
```

This type of issue is difficult because every symptom points to a different layer.

## 10. Initial Assumptions

At different stages, the issue appeared to be related to:

- DNS server issue
- Browser issue
- Website/server issue
- Firewall issue
- Internet access gateway issue
- Local portal issue
- User endpoint issue
- Public DNS issue
- Internal DNS record issue
- SSL/TLS issue
- UDP/QUIC filtering issue

The final understanding required combining all these layers.

## 11. Troubleshooting Process

The troubleshooting process was performed over multiple attempts and tests.

### 11.1 User-Side Testing

Affected users were tested with different browsers and DNS settings.

Checks included:

- Chrome access
- Firefox access
- DNS cache clear
- Static DNS testing
- Public DNS testing
- Local DNS testing
- Website access from different users
- Website access from different VLANs or networks

Observation:

```text
Issue behavior changed depending on browser and DNS path.
```

This showed that the problem was not only a single user machine issue.

### 11.2 DNS Testing

DNS resolution was tested using internal and public DNS paths.

Checks included:

```text
nslookup website.example.com
nslookup website.example.com <internal-dns-server>
nslookup website.example.com 8.8.8.8
```

Key finding:

```text
Internal DNS behavior was different because the internal AD DNS zone matched the public domain name.
```

This confirmed the split-DNS/design issue.

### 11.3 Browser Testing

Different browsers were tested.

Observation:

```text
Some portals behaved differently in Chrome compared to Firefox.
```

This indicated that browser-specific protocol behavior might be involved.

### 11.4 QUIC Testing

Chrome QUIC behavior was reviewed.

After disabling QUIC in Chrome, some local websites and portals started opening correctly.

This suggested that UDP/QUIC handling was involved.

### 11.5 Gateway and Protocol Filtering Review

The internet access gateway/security filtering policies were reviewed.

It was found that UDP-based traffic or required browser-related protocols were restricted.

After identifying and allowing the required protocols/traffic, the website access issue improved and became stable.

## 12. Root Cause

The root cause was not a single configuration issue.

It was a combination of DNS design and protocol filtering.

Main contributing factors:

```text
1. Internal AD DNS zone used the same root domain as public website domain
2. Internal users depended on internal DNS for domain resolution
3. Required public/local website records were not resolving correctly in all cases
4. Chrome QUIC behavior introduced UDP-based web traffic behavior
5. UDP/QUIC traffic was restricted on the internet access gateway
6. Required protocols for some websites/browsers were not fully allowed
```

Because of this combination, website access became inconsistent across users, browsers, and DNS configurations.

## 13. Root Cause Summary

```text
DNS design:
✘ AD/internal DNS zone used root public domain

Website resolution:
✘ Some public/local records were not handled correctly internally

Browser behavior:
✘ Chrome used QUIC/UDP behavior for some traffic

Gateway filtering:
✘ UDP/QUIC or required protocols were restricted

Result:
✘ Local websites and portals failed for many users
```

The issue was not simply “DNS down.”

The issue was a multi-layer DNS, browser, and security gateway problem.

## 14. Fix Applied

The fix was applied in stages.

### 14.1 DNS Workaround / Record Handling

Because the AD domain design could not be changed easily in a large production environment, the DNS behavior had to be handled carefully.

Possible actions included:

- Reviewing internal DNS records
- Adding required website records internally where needed
- Testing public vs internal resolution
- Avoiding unnecessary DNS changes for users
- Keeping required local portal records available
- Managing split-DNS behavior carefully

### 14.2 Browser Protocol Testing

Chrome QUIC was disabled during testing, which helped identify the browser/protocol layer.

This confirmed that the issue was not only DNS.

### 14.3 Gateway Policy Correction

Required protocols and web traffic were reviewed on the internet access gateway/security device.

UDP/QUIC-related blocking and required browser/web protocols were adjusted where needed.

After allowing the required traffic, affected websites and portals became accessible more reliably.

## 15. Verification After Fix

After applying the fixes/workarounds, the following verification was performed:

- Tested affected websites from multiple users
- Tested local portals from Chrome
- Tested local portals from Firefox
- Tested DNS resolution through internal DNS
- Tested access without repeated DNS manual changes
- Verified gateway policy behavior
- Verified required protocol access
- Confirmed improved website stability
- Confirmed users could access required portals

Result:

```text
Local website access: Restored
Portal access: Improved/stable
Chrome behavior: Improved after protocol handling
DNS behavior: Managed through internal records/workarounds
Gateway filtering: Corrected for required protocols
User impact: Reduced/resolved
```

## 16. Technical Explanation

This case had two major technical layers.

### Layer 1: Split-DNS / Internal DNS Conflict

When Active Directory is created using the same name as a public website domain, internal DNS becomes authoritative for that domain.

For example:

```text
Internal AD DNS zone:
example.com

Public website:
example.com
```

Internal DNS may not forward unresolved records for that zone to public DNS because it believes it owns the zone.

That means public website records must be handled internally if internal users need to access them.

This is why using a subdomain such as `ad.example.com` or `corp.example.com` for AD is often cleaner than using the root public domain directly.

### Layer 2: Browser Protocol and QUIC/UDP

Modern browsers may use different protocols and transport behavior.

Chrome may use QUIC/UDP-based behavior for some connections.

If a security gateway blocks or mishandles UDP/QUIC traffic, the browser may fail or behave inconsistently.

This can make a website issue look like:

```text
DNS issue
Browser issue
Firewall issue
Website issue
```

when it is actually a combination of these.

## 17. Useful Checks

Useful DNS checks:

```text
nslookup website.example.com
nslookup website.example.com <internal-dns-server>
nslookup website.example.com 8.8.8.8
ipconfig /flushdns
ipconfig /displaydns
```

Useful browser checks:

```text
Test in Chrome
Test in Firefox
Clear browser cache
Test with QUIC disabled
Test from incognito/private mode
Check browser error message
```

Useful network/security checks:

```text
Check firewall/IAG logs
Check UDP traffic policy
Check QUIC-related behavior
Check allowed web protocols
Check DNS traffic logs
Check SSL/TLS inspection if used
Check policy hit count
Check deny/drop logs
```

Useful validation questions:

```text
Is internal DNS authoritative for the same public domain?
Does the internal DNS contain the required website records?
Does public DNS return a different answer?
Does the issue happen in all browsers or only one browser?
Does disabling QUIC change behavior?
Is UDP/443 or related UDP traffic blocked?
Is the gateway filtering required web protocols?
Are users in different VLANs affected differently?
```

## 18. Prevention Recommendations

To avoid similar issues in future environments:

- Avoid using the root public domain directly as the AD domain name
- Use a subdomain such as `ad.example.com`, `corp.example.com`, or `internal.example.com`
- Maintain proper split-DNS records for public services used internally
- Document public vs internal DNS resolution paths
- Test websites from internal DNS and public DNS
- Include browser protocol behavior in web access troubleshooting
- Review QUIC/UDP handling on gateways and firewalls
- Avoid random per-user DNS changes without root cause analysis
- Standardize browser/network troubleshooting checklist
- Monitor gateway deny logs during website access issues

## 19. Key Lessons

- Website access issues are not always simple DNS issues.
- Internal AD DNS design can affect public website resolution.
- Using the same root domain internally and publicly can create split-DNS challenges.
- Browser behavior matters during troubleshooting.
- Chrome and Firefox may behave differently because of protocol differences.
- QUIC/UDP filtering can break or affect some web traffic.
- Changing DNS randomly may hide the symptom but not fix the design issue.
- Complex issues often require DNS, firewall, browser, and gateway teams to troubleshoot together.

## 20. One-Line Takeaway

The websites were not simply down; access was affected by a combination of internal DNS domain design, split-DNS behavior, Chrome QUIC/UDP traffic, and gateway protocol filtering.
