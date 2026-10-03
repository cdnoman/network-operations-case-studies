# Firewall HA Case Study: Ghost MAC Address Blocking HA Interface Failover

> Note: This case study is sanitized for public sharing. Organization names, real IP addresses, hostnames, interface names, screenshots, and sensitive topology details have been removed, changed, or generalized.

## 1. Overview

This case study documents a firewall High Availability troubleshooting scenario where HA behavior was affected due to a ghost/stale MAC address entry in the network path.

The firewall HA pair was expected to provide failover between the primary and secondary firewall. However, during testing and troubleshooting, traffic behavior was not normal after failover. The expected firewall unit was active, but some traffic was still not passing correctly.

The issue was later traced to a stale MAC address entry on the connected switching path, which caused traffic to follow the wrong Layer 2 forwarding path.

## 2. Environment

The environment included:

- Firewall HA pair
- Core/distribution switching
- Trunk/access connectivity toward firewall interfaces
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
