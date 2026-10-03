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
