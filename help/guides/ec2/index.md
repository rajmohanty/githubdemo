---
layout: help-security
title:  Custom Security Group Management
slug:   custom-security-group-management
date:   2016-08-17 09:00:00
tags:
- Security
- EC2
- AWS
breadcrumbs:
- Security
- Help
---

There are a few different approaches teams can take to manage custom security
groups through Turbot and AWS.

**Default Security Group:**

Default security groups are managed in Turbot through the [Turbot Networks
Admin Page](/admin/networks).  Per Network, Turbot Admins can associate a
default security group that would be applied to all VPCs associated in that
network.  This allows for an easy to manage and consistent default security
group across all VPCs in the same network definition.

**Custom Egress Rules per VPC:**

Within the [Turbot Networks Admin Page](/admin/networks), Turbot Admins can
create egress rules per VPC from the option Custom Security Group Rules >
Create Security Group Rule.   This allows Admins to quickly assign an egress
rule to the applicable VPC that is an addition to the default security group.
Turbot will add the rules specifically to that VPCs default security group.

**Custom Security Groups per Account:**

Turbot Admins can allow for users with AWS/Admin or EC2/Admin permissions to
manage their own security groups directly in AWS across one or many AWS
accounts.  Turbot Admins can enable this option through their EC2 Options >
Custom Security Group Management.  Once enabled, Turbot Admins can also define
ingress and egress boundaries they want to restrict security groups to:

* Custom Security Group Restrict To Zone - Custom security group ingress rules
  may only be added within the specified zone.
* Custom Security Group Minimum Bitmask - Defines the smallest allowed bitmask
  (largest range of IP addresses) that can be granted access in any single
  custom security group ingress rule.
* Custom Security Group Restrict Maximum Port Range - The maximum number of
  ports that may be opened by any single ingress rule in a security group.
* Custom Security Group Blacklisted Ports - A YAML list of ports that are
  blacklisted and may not be used for ingress in custom security groups.

**Custom Security Groups per Account using AWS/SuperUser:**

As an Administrative backend approach when the above approaches are not
appropriate, a Turbot Admin with AWS/SuperUser rights can login to the
applicable Account with AWS SuperUser from the [Turbot Accounts Admin
Page](/admin/accounts) to create custom security groups directly.