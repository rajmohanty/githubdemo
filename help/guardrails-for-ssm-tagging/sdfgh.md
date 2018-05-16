---
layout: help-security
title:  Guardrails for AWS Elastic Compute Cloud (EC2)
slug:   guardrails-for-aws-ec2
date:   2016-02-21 09:00:00
tags:
- Security
- EC2
- ELB
- Auto Scaling
- Server
- AWS
breadcrumbs:
- Security
- Help
---

## Introduction

Amazon Elastic Compute Cloud (Amazon EC2) is a web service that provides
resizable compute capacity in the cloud.  It is designed to make web-scale
[cloud computing](https://aws.amazon.com/what-is-cloud-computing/) easier for
developers.

Turbot provides enterprise guardrails for AWS, including secure configuration
and management of EC2 instances including Autoscaling (ASG), Elastic Load
Balancing (ELB), and Elastic Block Storage (EBS) capabilities

This guide outlines the security controls implemented by Turbot and is intended
for review and discussion with security and application teams to establish
appropriate policies and controls based on the needs of the organization.


## Shared Responsibility

EC2 is run under the [AWS shared responsibility
model](http://aws.amazon.com/compliance/shared-responsibility-model/). Security
“of the cloud”, provided by AWS for EC2, is documented and covered by their
many [compliance certifications](http://aws.amazon.com/compliance/).

Turbot provides automations, policies and controls to help organizations
deliver security “in the cloud”.


## Network Security for EC2 Instances - Default Groups

Security of and operational controls for the virtual network infrastructure are
provided by AWS.

Configuration of the network is managed by Turbot, including:

* Application environments are separated into different VPCs.
* Each VPC includes at least two subnets, in separate availability zones.
* Security groups, providing host level firewalls, are provided for standard
  application and web tiers.

EC2 instances can only be started inside a Turbot managed VPC. EC2 instances
run inside a VPC designated for the Turbot Account, and may be started in one
of two availability zones.

Each EC2 instance runs with one or many specific Security Groups, opening only
access to the ports appropriate for the application workload.

The default Security Group ingress and egress configurations are managed at the
Cluster level.  Note: All EC2 instances should associate the default security
group for core services.

Below are OOTB security groups created specifically for EC2 use:

<table class="table table-bordered table-hover">
<tr><th>Engine</th><th>Security Group</th><th>Port</th><th>Bastion</th><th>Intranet</th><th>VPC</th></tr>

<tr>
  <td rowspan="4">Application</td>
  <td rowspan="2">AppVPC</td>
  <td>8080-8083</td>
  <td>Bastion</td>
  <td></td>
  <td>VPC</td>
</tr>
<tr>
  <td>8443-8446</td>
  <td>Bastion</td>
  <td></td>
  <td>VPC</td>
</tr>
<tr>
  <td rowspan="2">AppDMA</td>
  <td>8080-8083</td>
  <td>Bastion</td>
  <td>Intranet</td>
  <td>VPC</td>
</tr>
<tr>
  <td>8443-8446</td>
  <td>Bastion</td>
  <td>Intranet</td>
  <td>VPC</td>
</tr>

<tr>
  <td rowspan="4">Web</td>
  <td rowspan="2">WebVPC</td>
  <td>80</td>
  <td>Bastion</td>
  <td></td>
  <td>VPC</td>
</tr>
<tr>
  <td>443</td>
  <td>Bastion</td>
  <td></td>
  <td>VPC</td>
</tr>
<tr>
  <td rowspan="2">WebDMA</td>
  <td>80</td>
  <td>Bastion</td>
  <td>Intranet</td>
  <td>VPC</td>
</tr>
<tr>
  <td>443</td>
  <td>Bastion</td>
  <td>Intranet</td>
  <td>VPC</td>
</tr>

</table>


## Network Security for EC2 Instances - Custom Groups

Beyond the default security groups that are created in the account by Turbot.
There are additional options that allow an account to create custom security
groups in addition to the default groups.

* Security Group AllVPC Enabled - If enabled, the AllVPC security group is
  available for use. This security group opens all ports on the receiving
  server for access from other servers in the VPC.
* Custom Security Group Management - If enabled, users with the EC2/Admin role
  can manage security groups through AWS and subject to the guardrail
  definitions from the following options.
  * Restrict to Zone - If enabled, restricts ingress rules that can only apply
    to a specific zone (e.g. Intranet, VPC, Anywhere).
  * Restrict Minimum Bitmask - If enabled, defines the smallest allowed bitmask
    (largest range of IP addresses) that can be granted access in any single
    custom security group.
  * Restrict Maximum Port Range - If enabled, the maximum number of ports that
    may be opened by any single rule in a security group.
  * Blacklist Ports - a list of ports can be inputted in a yaml formatted list
    (a dash and a space) that may not be used in a security group.


## Server Infrastructure Access & Permissions in EC2

Turbot establishes IAM groups in AWS that support least privilege and
separation of duties.

Groups specifically created for EC2 are:

* AWS/EC2/Admin – Medium to high risk operations (e.g. delete instance).
* AWS/EC2/Operator – Low to medium risk operations (e.g. reboot instance).
* AWS/EC2/Metadata – View EC2 instance configurations only.

Turbot also provides access to EC2 through AWS higher level roles:

* AWS/Admin – Includes AWS/EC2/Admin.
* AWS/Operator – Includes AWS/EC2/Operator.
* AWS/Metadata – Includes AWS/EC2/Metadata.

Turbot provides these permissions at both the Account specific, and Cluster
wide levels. Users granted the role at the Cluster level can temporarily
elevate their permissions to assume the role in a specific account.

All permissions granted in AWS EC2 relate only to server infrastructure
management, and provide no privileges within the instance to the Operating
System.

See Appendix A for the full Turbot permission mapping for EC2.


## Encryption at rest

AWS [EC2 supports encryption at rest for EBS
volumes](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html).

Turbot can enforce encryption at rest using the Encryption at Rest option.  If
enabled, Turbot will detect any unencrypted EBS volumes and immediately raise
an alarm. Turbot does not delete or quarantine the instance as data migration
may be required.  Note: EBS volume encryption has negligible impact on
performance, but requires specific instance types, cannot be used for boot
volumes and cannot be changed for existing volumes.


## Data Protection

Turbot can enforce a backup retention period policy for all server instances
through setting the EC2 Snapshot Retention Period option. This setting protects
organizational data from accidental instance and volume deletion without
sufficient backups.

The Snapshot Period in Hours option can be set from 0 - 168 hours.  If the
number is set to zero, Turbot will not automate volume backups.  All snapshots
are highly available; spread across multiple availability zones in a region.


## AMI, Image, Operating System Management

Turbot can enforce if accounts are allowed to create, manage, and publish AMIs.
These configurations are enabled through the following options:

* Allow Local AMIs - Allow creation and management of AMIs in the account.
* Allow AMI Publishing - Allow AMIs owned by the account to be shared with
  other accounts.

To have Turbot enforce what AMIs or Images can be used within an account, the
following configurations can be enabled:

* Allow Local AMIs - Allow AMIs owned by the account to be run in the account.
* Trusted AMI Publishers - Allow AMIs owned by other accounts in this list to
  be run in the account.
* Current AMIs - A list of approved AMI IDs that Turbot will allow to be used
  in the account.
* Deprecated AMIs - Will allow existing instances to remain provisioned,
  however new instances cannot be created of the AMI IDs in the list
* Allow DMZ Instances - Allow EC2 instances to be launched in DMZ subnets

Turbot also manages Operating System configurations on AWS Linux, Ubuntu 12.04
& 14.04, and RHEL 7 images.  The following configurations can be enabled:

* Linux CIS Hardening - If enabled, then Turbot will apply CIS hardening to
  Linux instances launched with the turbot key pair.
* Linux User Management - If enabled, then Turbot will manage user accounts for
  Linux instances based on membership in the Linux/* roles. Applies only to
  Linux instances launched with the turbot key pair.
* Linux Environment Management - If enabled, then Turbot will manage
  environment settings for Linux instances such as timezone, DNS, etc. Applies
  only to Linux instances launched with the turbot key pair.
* Linux Patch Management - If enabled, Turbot will setup regular security
  patching for Linux instances. Applies only to Linux instances launched with
  the turbot key pair.


## Elastic IP Addresses

Elastic IP addresses are static IP addresses which can be associated with EC2
instances. Instances that have an associated elastic IP address are publicly
accessible ONLY if routing is configured for direct Internet access through an
IGW. Management of elastic IP addresses is available to users with
AWS/EC2/Admin permissions.


## Infrastructure Health Alarms

Turbot automatically applied best practice infrastructure health alarms through
AWS Cloudwatch to any new EC2 instance.

The following alarms are automatically applied to each new instance:
* Status Check Failed - EC2 Instance has been Unhealthy for 10 Minutes
* CPU Utilization - EC2 Instance CPU Utilization has reached greater than 85%
  for 60 Minutes


## Turbot EC2 Configuration Alarms

Turbot runs many configuration tasks in the background automatically to ensure
policies are being enforced and good configurations are being met.

In addition to managing the policy options available above, Turbot also will
alarm the account if any of the following are misconfigured in EC2:
* Instance Allowed - if the EC2 Instance exists in an account where the EC2
  service is not enabled.
* Hostname Tag - adds the hostname of the EC2 Instance automatically to a
  Hostname tag on the instance


## User Activity & Audit Logging

AWS CloudTrail supports logging of all [EC2 actions as an audit
trail](http://docs.aws.amazon.com/AWSEC2/latest/APIReference/using-cloudtrail.html).
Turbot ensures that CloudTrail is enabled for all user access in all regions of
all accounts.


## Appendix A – Turbot permission levels for AWS EC2

<table class="table table-bordered table-hover">
<tr><th>AWS IAM Permission</th><th>Turbot Level</th><th>Notes</th></tr>
<tr><td>acm:ListCertificates</td><td>Metadata</td><td>Required for ELB launches.</td></tr>
<tr><td>application-autoscaling:DeleteScalingPolicy</td><td>Admin</td><td>Admins can change the autoscaling process.</td></tr>
<tr><td>application-autoscaling:DeleteScheduledAction</td><td>Admin</td><td></td></tr>
<tr><td>application-autoscaling:DeregisterScalableTarget</td><td>Operator</td><td></td></tr>
<tr><td>application-autoscaling:DescribeScalableTargets</td><td>Metadata</td><td></td></tr>
<tr><td>application-autoscaling:DescribeScalingActivities</td><td>Metadata</td><td></td></tr>
<tr><td>application-autoscaling:DescribeScalingPolicies</td><td>Metadata</td><td></td></tr>
<tr><td>application-autoscaling:DescribeScheduledActions</td><td>Metadata</td><td></td></tr>
<tr><td>application-autoscaling:PutScalingPolicy</td><td>Admin</td><td>Admins can change the autoscaling process.</td></tr>
<tr><td>application-autoscaling:PutScheduledAction</td><td>Admin</td><td></td></tr>
<tr><td>application-autoscaling:RegisterScalableTarget</td><td>Operator</td><td></td></tr>
<tr><td>autoscaling:AttachInstances</td><td>Whitelist</td><td>Operators can manage instances in an autoscaling group but not change its config.</td></tr>
<tr><td>autoscaling:AttachLoadBalancerTargetGroups</td><td>Whitelist</td><td>Operators can manage instances in an autoscaling group but not change its config.</td></tr>
<tr><td>autoscaling:AttachLoadBalancers</td><td>Whitelist</td><td>Operators can manage instances in an autoscaling group but not change its config.</td></tr>
<tr><td>autoscaling:CompleteLifecycleAction</td><td>Whitelist</td><td>Allows custom steps in the autoscaling lifecycle process</td></tr>
<tr><td>autoscaling:CreateAutoScalingGroup</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:CreateLaunchConfiguration</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:CreateOrUpdateScalingTrigger</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:CreateOrUpdateTags</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:CreateScalingPlan</td><td>Whitelist</td><td>Admins can create autoscaling plan.</td></tr>
<tr><td>autoscaling:DeleteAutoScalingGroup</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:DeleteLaunchConfiguration</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:DeleteLifecycleHook</td><td>Whitelist</td><td>Admins can change the autoscaling process.</td></tr>
<tr><td>autoscaling:DeleteNotificationConfiguration</td><td>Whitelist</td><td>Operators can control monitoring & notification of the autoscaling group.</td></tr>
<tr><td>autoscaling:DeletePolicy</td><td>Whitelist</td><td>Admins can change the autoscaling process.</td></tr>
<tr><td>autoscaling:DeleteScalingPlan</td><td>Whitelist</td><td>Admins can delete autoscaling plan.</td></tr>
<tr><td>autoscaling:DeleteScheduledAction</td><td>Whitelist</td><td>Admins can change the autoscaling process.</td></tr>
<tr><td>autoscaling:DeleteTags</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:DeleteTrigger</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:DescribeAccountLimits</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeAdjustmentTypes</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeAutoScalingGroups</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeAutoScalingInstances</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeAutoScalingNotificationTypes</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeLaunchConfigurations</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeLifecycleHookTypes</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeLifecycleHooks</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeLoadBalancerTargetGroups</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeLoadBalancers</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeMetricCollectionTypes</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeNotificationConfigurations</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribePolicies</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeScalingActivities</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeScalingPlanResources</td><td>Metadata</td><td>Describes the scalable resources in the specified scaling plan.</td></tr>
<tr><td>autoscaling:DescribeScalingPlans</td><td>Metadata</td><td>Describes the specified scaling plans or all of your scaling plans.</td></tr>
<tr><td>autoscaling:DescribeScalingProcessTypes</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeScheduledActions</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeTags</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeTerminationPolicyTypes</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DescribeTriggers</td><td>Metadata</td><td></td></tr>
<tr><td>autoscaling:DetachInstances</td><td>Whitelist</td><td>Operators can manage instances in an autoscaling group but not change its config.</td></tr>
<tr><td>autoscaling:DetachLoadBalancerTargetGroups</td><td>Whitelist</td><td>Operators can manage instances in an autoscaling group but not change its config.</td></tr>
<tr><td>autoscaling:DetachLoadBalancers</td><td>Whitelist</td><td>Operators can manage instances in an autoscaling group but not change its config.</td></tr>
<tr><td>autoscaling:DisableMetricsCollection</td><td>Whitelist</td><td>Operators can control monitoring & notification of the autoscaling group.</td></tr>
<tr><td>autoscaling:EnableMetricsCollection</td><td>Whitelist</td><td>Operators can control monitoring & notification of the autoscaling group.</td></tr>
<tr><td>autoscaling:EnterStandby</td><td>Whitelist</td><td>Operators can manage instances in an autoscaling group but not change its config.</td></tr>
<tr><td>autoscaling:ExecutePolicy</td><td>Whitelist</td><td>Operators can execute a policy that was defined by an Admin.</td></tr>
<tr><td>autoscaling:ExitStandby</td><td>Whitelist</td><td>Operators can manage instances in an autoscaling group but not change its config.</td></tr>
<tr><td>autoscaling:PutLifecycleHook</td><td>Whitelist</td><td>Admins can change the autoscaling process.</td></tr>
<tr><td>autoscaling:PutNotificationConfiguration</td><td>Whitelist</td><td>Operators can control monitoring & notification of the autoscaling group.</td></tr>
<tr><td>autoscaling:PutScalingPolicy</td><td>Whitelist</td><td>Admins can change the autoscaling process.</td></tr>
<tr><td>autoscaling:PutScheduledUpdateGroupAction</td><td>Whitelist</td><td>Admins can change the autoscaling process.</td></tr>
<tr><td>autoscaling:RecordLifecycleActionHeartbeat</td><td>Whitelist</td><td>Operators can manage instances in an autoscaling group but not change its config.</td></tr>
<tr><td>autoscaling:ResumeProcesses</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:SetDesiredCapacity</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:SetInstanceHealth</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:SetInstanceProtection</td><td>Whitelist</td><td>Operators can manage instances in an autoscaling group but not change its config.</td></tr>
<tr><td>autoscaling:SuspendProcesses</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:TerminateInstanceInAutoScalingGroup</td><td>Whitelist</td><td></td></tr>
<tr><td>autoscaling:UpdateAutoScalingGroup</td><td>Whitelist</td><td></td></tr>
<tr><td>aws-marketplace:BatchMeterUsage</td><td>Admin</td><td>Administrators may report software usage.</td></tr>
<tr><td>aws-marketplace:GetEntitlements</td><td>Metadata</td><td>Viewing entitlement of a customer to a given product. http://docs.aws.amazon.com/marketplaceentitlement/latest/APIReference/Welcome.html.</td></tr>
<tr><td>aws-marketplace:MeterUsage</td><td>Admin</td><td>Administrators may report software usage.</td></tr>
<tr><td>aws-marketplace:ResolveCustomer</td><td>Admin</td><td>Used by SaaS application and returns customer identifier and product code based on registration token.</td></tr>
<tr><td>aws-marketplace:Subscribe</td><td>Whitelist</td><td>Administrators may subscribe to marketplace software.</td></tr>
<tr><td>aws-marketplace:Unsubscribe</td><td>Whitelist</td><td>Administrators may subscribe to marketplace software.</td></tr>
<tr><td>aws-marketplace:ViewSubscriptions</td><td>Metadata</td><td>Viewing marketplace subscriptions is required for server management if the marketplace is used.</td></tr>
<tr><td>cloudwatch:DescribeAlarmHistory</td><td>Metadata</td><td>For console access per EC2 ReadOnly policy</td></tr>
<tr><td>cloudwatch:DescribeAlarms</td><td>Metadata</td><td>For console access per EC2 ReadOnly policy</td></tr>
<tr><td>cloudwatch:DescribeAlarmsForMetric</td><td>Metadata</td><td>For console access per EC2 ReadOnly policy</td></tr>
<tr><td>cloudwatch:GetMetricData</td><td>Metadata</td><td>This allows GetMetricData API to retrieve as many as metrics data and to perform mathematical expressions on this data.</td></tr>
<tr><td>cloudwatch:GetMetricStatistics</td><td>Metadata</td><td>For console access per EC2 ReadOnly policy</td></tr>
<tr><td>cloudwatch:ListMetrics</td><td>Metadata</td><td>For console access per EC2 ReadOnly policy</td></tr>
<tr><td>ec2-reports:ViewInstanceUsageReport</td><td>Metadata</td><td>Obscure permission http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/usage-reports.html#iam-access-ec2-reports</td></tr>
<tr><td>ec2-reports:ViewReservedInstanceUtilizationReport</td><td>Metadata</td><td>Obscure permission http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/usage-reports.html#iam-access-ec2-reports</td></tr>
<tr><td>ec2:AcceptReservedInstancesExchangeQuote</td><td>Admin</td><td>Accounts can manage their own reserved instances (but cannot resell them).</td></tr>
<tr><td>ec2:ActivateLicense</td><td>None</td><td>Not currently in use by AWS - http://aws.amazon.com/blogs/aws/bring_your_own_ea_windows_server_license_to_ec2/</td></tr>
<tr><td>ec2:AllocateAddress</td><td>Admin</td><td>Admins can allocate new elastic IP addresses; this is considered safe as the proper routing still needs to be configured for public access.</td></tr>
<tr><td>ec2:AllocateHosts</td><td>Admin</td><td></td></tr>
<tr><td>ec2:AssignIpv6Addresses</td><td>Admin</td><td>Private IP addresses are within the allocated space to the account so can be safely managed by the account.</td></tr>
<tr><td>ec2:AssignPrivateIpAddresses</td><td>Admin</td><td>Private IP addresses are within the allocated space to the account so can be safely managed by the account.</td></tr>
<tr><td>ec2:AssociateAddress</td><td>Admin</td><td>Admins can associate elastic IP addresses; this is considered safe as the proper routing still needs to be configured for public access.</td></tr>
<tr><td>ec2:AssociateIamInstanceProfile</td><td>Admin</td><td>Admins manage IAM instance profile associations for existing instances.</td></tr>
<tr><td>ec2:AttachNetworkInterface</td><td>Admin</td><td>Network interfaces can be safely used by the account inside the VPC context.</td></tr>
<tr><td>ec2:AttachVolume</td><td>Admin</td><td></td></tr>
<tr><td>ec2:AuthorizeSecurityGroupEgress</td><td>Whitelist</td><td></td></tr>
<tr><td>ec2:AuthorizeSecurityGroupIngress</td><td>Whitelist</td><td></td></tr>
<tr><td>ec2:BundleInstance</td><td>Whitelist</td><td>Optional VM export.</td></tr>
<tr><td>ec2:CancelBundleTask</td><td>Whitelist</td><td>Optional VM export.</td></tr>
<tr><td>ec2:CancelConversionTask</td><td>Whitelist</td><td>Optional VM export.</td></tr>
<tr><td>ec2:CancelExportTask</td><td>Whitelist</td><td>Optional VM export.</td></tr>
<tr><td>ec2:CancelImportTask</td><td>Whitelist</td><td>Optional VM import.</td></tr>
<tr><td>ec2:CancelReservedInstancesListing</td><td>None</td><td>Reserved instance reselling is at the cluster level.</td></tr>
<tr><td>ec2:CancelSpotFleetRequests</td><td>Admin</td><td></td></tr>
<tr><td>ec2:CancelSpotInstanceRequests</td><td>Admin</td><td>Accounts can safely use spot instances within their VPC.</td></tr>
<tr><td>ec2:ConfirmProductInstance</td><td>None</td><td>Only relevant to AMI marketplace sellers.</td></tr>
<tr><td>ec2:CopyFpgaImage</td><td>Whitelist</td><td>Copies the specified Amazon FPGA Image (AFI) to the current region.</td></tr>
<tr><td>ec2:CopyImage</td><td>Whitelist</td><td>Optional image management. Copies image between regions not across accounts.</td></tr>
<tr><td>ec2:CopySnapshot</td><td>Operator</td><td>Low risk operation to copy data within the same account.</td></tr>
<tr><td>ec2:CreateFpgaImage</td><td>Whitelist</td><td>Optional FPGA image management. https://aws.amazon.com/ec2/instance-types/f1/</td></tr>
<tr><td>ec2:CreateImage</td><td>Whitelist</td><td>Optional image management. Creates a new AMI from an instance in the account.</td></tr>
<tr><td>ec2:CreateInstanceExportTask</td><td>Whitelist</td><td>Optional VM export.</td></tr>
<tr><td>ec2:CreateKeyPair</td><td>Admin</td><td>Administrators use key pairs when starting new instances. This should be moved to an option in the future when Turbot supports non-key pair based login.</td></tr>
<tr><td>ec2:CreateLaunchTemplate</td><td>Admin</td><td></td></tr>
<tr><td>ec2:CreateLaunchTemplateVersion</td><td>Admin</td><td></td></tr>
<tr><td>ec2:CreateNetworkInterface</td><td>Admin</td><td>Network interfaces can be safely used by the account inside the VPC context.</td></tr>
<tr><td>ec2:CreatePlacementGroup</td><td>Admin</td><td>Servers can be safely placed within cluster managed networks.</td></tr>
<tr><td>ec2:CreateReservedInstancesListing</td><td>None</td><td>Reserved instance reselling is at the cluster level.</td></tr>
<tr><td>ec2:CreateSecurityGroup</td><td>Whitelist</td><td></td></tr>
<tr><td>ec2:CreateSnapshot</td><td>Operator</td><td>Low risk operation to backup data within the same account.</td></tr>
<tr><td>ec2:CreateSpotDatafeedSubscription</td><td>Admin</td><td>Accounts can safely use spot instances within their VPC.</td></tr>
<tr><td>ec2:CreateTags</td><td>Operator</td><td>Tags are low risk for management in Turbot since accounts are the isolation boundary; not tags.</td></tr>
<tr><td>ec2:CreateVolume</td><td>Admin</td><td>Storage management is safe within the account.</td></tr>
<tr><td>ec2:DeactivateLicense</td><td>None</td><td>Not currently in use by AWS - http://aws.amazon.com/blogs/aws/bring_your_own_ea_windows_server_license_to_ec2/</td></tr>
<tr><td>ec2:DeleteFpgaImage</td><td>Whitelist</td><td>Deletes the specified Amazon FPGA Image.</td></tr>
<tr><td>ec2:DeleteKeyPair</td><td>Admin</td><td></td></tr>
<tr><td>ec2:DeleteLaunchTemplate</td><td>Admin</td><td></td></tr>
<tr><td>ec2:DeleteLaunchTemplateVersions</td><td>Admin</td><td></td></tr>
<tr><td>ec2:DeleteNetworkInterface</td><td>Admin</td><td>Network interfaces can be safely used by the account inside the VPC context.</td></tr>
<tr><td>ec2:DeletePlacementGroup</td><td>Admin</td><td></td></tr>
<tr><td>ec2:DeleteSecurityGroup</td><td>Whitelist</td><td></td></tr>
<tr><td>ec2:DeleteSnapshot</td><td>Admin</td><td>Deletion of snapshots is limited to Admin though creation is open to Operator.</td></tr>
<tr><td>ec2:DeleteSpotDatafeedSubscription</td><td>Admin</td><td></td></tr>
<tr><td>ec2:DeleteTags</td><td>Operator</td><td>Tags are low risk for management in Turbot since accounts are the isolation boundary; not tags. Most deletions are denied to operator but tags are a low risk management activity even for deletion.</td></tr>
<tr><td>ec2:DeleteVolume</td><td>Admin</td><td></td></tr>
<tr><td>ec2:DeregisterImage</td><td>Whitelist</td><td>Optional image management. Deregisters an image preventing further launches.</td></tr>
<tr><td>ec2:DescribeAccountAttributes</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeAddresses</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeAvailabilityZones</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeBundleTasks</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeClassicLinkInstances</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeConversionTasks</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeElasticGpus</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeExportTasks</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeFpgaImageAttribute</td><td>Metadata</td><td>Describes the specified attribute of the specified Amazon FPGA Image.</td></tr>
<tr><td>ec2:DescribeFpgaImages</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeHostReservationOfferings</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeHostReservations</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeHosts</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeIamInstanceProfileAssociations</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeIdentityIdFormat</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeIdFormat</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeImageAttribute</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeImages</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeImportImageTasks</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeImportSnapshotTasks</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeInstanceAttribute</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeInstances</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeInstanceStatus</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeKeyPairs</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeLaunchTemplates</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeLaunchTemplateVersions</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeLicenses</td><td>Metadata</td><td>Note: Not currently in use by AWS - http://aws.amazon.com/blogs/aws/bring_your_own_ea_windows_server_license_to_ec2/</td></tr>
<tr><td>ec2:DescribeMovingAddresses</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeNetworkInterfaceAttribute</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeNetworkInterfacePermissions</td><td>Metadata</td><td>Describes the permissions of network interfaces.</td></tr>
<tr><td>ec2:DescribeNetworkInterfaces</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribePlacementGroups</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeRegions</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeReservedInstances</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeReservedInstancesListings</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeReservedInstancesModifications</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeReservedInstancesOfferings</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeScheduledInstanceAvailability</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeScheduledInstances</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeSecurityGroupReferences</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeSecurityGroups</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeSnapshotAttribute</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeSnapshots</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeSpotDatafeedSubscription</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeSpotFleetInstances</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeSpotFleetRequestHistory</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeSpotFleetRequests</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeSpotInstanceRequests</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeSpotPriceHistory</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeStaleSecurityGroups</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeTags</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeVolumeAttribute</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeVolumes</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeVolumesModifications</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DescribeVolumeStatus</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:DetachClassicLinkVpc</td><td>None</td><td>EC2 classic should not be used</td></tr>
<tr><td>ec2:DetachNetworkInterface</td><td>Admin</td><td>Network interfaces can be safely used by the account inside the VPC context.</td></tr>
<tr><td>ec2:DetachVolume</td><td>Admin</td><td></td></tr>
<tr><td>ec2:DisassociateAddress</td><td>Admin</td><td>Admins can disassociate elastic IP addresses.</td></tr>
<tr><td>ec2:DisassociateIamInstanceProfile</td><td>Admin</td><td>Admins manage IAM instance profile associations for existing instances.</td></tr>
<tr><td>ec2:EnableVolumeIO</td><td>Admin</td><td></td></tr>
<tr><td>ec2:GetConsoleOutput</td><td>Metadata</td><td>Allows viewing of console data from machines; helpful for monitoring & investigating system during bootup. Considered to be metadata not ReadOnly since systems should not be logging sensitive information and it's a key part of troubleshooting before needing access to the actual machine (which would obviously be at least ReadOnly).</td></tr>
<tr><td>ec2:GetConsoleScreenshot</td><td>Metadata</td><td>Allows viewing of on-demand screenshot of instance console for machines which is helpful for monitoring & investigating systems when they become unreachable via RDS and SSH. Considered to be metadata and not ReadOnly since it's a key part of troubleshooting when the instance is unreachable.</td></tr>
<tr><td>ec2:GetHostReservationPurchasePreview</td><td>Metadata</td><td>Allows preview of host reservation purchase but does not result in offering being purchased.</td></tr>
<tr><td>ec2:GetLaunchTemplateData</td><td>Metadata</td><td>Launch template data contains metadata about the EC2 instance.</td></tr>
<tr><td>ec2:GetPasswordData</td><td>Admin</td><td>Required for launch of Windows machines and used with key pairs.</td></tr>
<tr><td>ec2:GetReservedInstancesExchangeQuote</td><td>Metadata</td><td></td></tr>
<tr><td>ec2:ImportImage</td><td>Whitelist</td><td>Optional VM import.</td></tr>
<tr><td>ec2:ImportInstance</td><td>Whitelist</td><td>Optional VM import.</td></tr>
<tr><td>ec2:ImportKeyPair</td><td>Admin</td><td>Administrators use key pairs when starting new instances. This should be moved to an option in the future when Turbot supports non-key pair based login.</td></tr>
<tr><td>ec2:ImportSnapshot</td><td>Whitelist</td><td>Optional VM import.</td></tr>
<tr><td>ec2:ImportVolume</td><td>Whitelist</td><td>Optional VM import.</td></tr>
<tr><td>ec2:ModifyFpgaImageAttribute</td><td>Whitelist</td><td>Modifies the specified attributes(description | name | loadPermission | productCodes) of the specified Amazon FPGA Image.</td></tr>
<tr><td>ec2:ModifyHosts</td><td>Admin</td><td></td></tr>
<tr><td>ec2:ModifyIdentityIdFormat</td><td>Admin</td><td></td></tr>
<tr><td>ec2:ModifyIdFormat</td><td>Admin</td><td></td></tr>
<tr><td>ec2:ModifyImageAttribute</td><td>Whitelist</td><td>Optional image attribute management.</td></tr>
<tr><td>ec2:ModifyInstanceAttribute</td><td>Admin</td><td>Accounts can manage their own instances.</td></tr>
<tr><td>ec2:ModifyInstancePlacement</td><td>Admin</td><td></td></tr>
<tr><td>ec2:ModifyLaunchTemplate</td><td>Admin</td><td></td></tr>
<tr><td>ec2:ModifyNetworkInterfaceAttribute</td><td>Admin</td><td>Network interfaces can be safely used by the account inside the VPC context.</td></tr>
<tr><td>ec2:ModifyReservedInstances</td><td>Admin</td><td>Accounts can manage their own reserved instances (but cannot resell them).</td></tr>
<tr><td>ec2:ModifySnapshotAttribute</td><td>Admin</td><td>Allows for cross-account access.</td></tr>
<tr><td>ec2:ModifySpotFleetRequest</td><td>Admin</td><td></td></tr>
<tr><td>ec2:ModifyVolume</td><td>Admin</td><td>Within account storage performance changes.</td></tr>
<tr><td>ec2:ModifyVolumeAttribute</td><td>Admin</td><td>Within account storage performance changes.</td></tr>
<tr><td>ec2:MonitorInstances</td><td>Admin</td><td>Monitoring frequency is managed by accounts.</td></tr>
<tr><td>ec2:MoveAddressToVpc</td><td>None</td><td>EC2 classic should not be used</td></tr>
<tr><td>ec2:PurchaseHostReservation</td><td>Admin</td><td>Accounts can manage their own dedicated hosts.</td></tr>
<tr><td>ec2:PurchaseReservedInstancesOffering</td><td>Admin</td><td>Accounts can manage their own reserved instances (but cannot resell them).</td></tr>
<tr><td>ec2:PurchaseScheduledInstances</td><td>Owner</td><td>Long term subscriptions are managed by owners.</td></tr>
<tr><td>ec2:RebootInstances</td><td>Operator</td><td>Operators can start stop and reboot existing instances.</td></tr>
<tr><td>ec2:RegisterImage</td><td>Whitelist</td><td>Optional image management. Registers an AMI for launching; typically done automatically as part of CreateImage.</td></tr>
<tr><td>ec2:ReleaseAddress</td><td>Admin</td><td>Admins can release elastic IP addresses.</td></tr>
<tr><td>ec2:ReleaseHosts</td><td>Admin</td><td></td></tr>
<tr><td>ec2:ReplaceIamInstanceProfileAssociation</td><td>Admin</td><td>Admins manage IAM instance profile associations for existing instances.</td></tr>
<tr><td>ec2:ReportInstanceStatus</td><td>Operator</td><td>Operators can report bad instances to AWS support.</td></tr>
<tr><td>ec2:RequestSpotFleet</td><td>Admin</td><td></td></tr>
<tr><td>ec2:RequestSpotInstances</td><td>Admin</td><td>Accounts can safely use spot instances within their VPC.</td></tr>
<tr><td>ec2:ResetFpgaImageAttribute</td><td>Whitelist</td><td>Resets the the load permission attribute.</td></tr>
<tr><td>ec2:ResetImageAttribute</td><td>Whitelist</td><td>Optional image attribute management.</td></tr>
<tr><td>ec2:ResetInstanceAttribute</td><td>Admin</td><td>Instances are managed by accounts.</td></tr>
<tr><td>ec2:ResetNetworkInterfaceAttribute</td><td>None</td><td>Network interfaces can be safely used by the account inside the VPC context.</td></tr>
<tr><td>ec2:ResetSnapshotAttribute</td><td>Admin</td><td></td></tr>
<tr><td>ec2:RestoreAddressToClassic</td><td>None</td><td>EC2 classic should not be used</td></tr>
<tr><td>ec2:RevokeSecurityGroupEgress</td><td>Whitelist</td><td></td></tr>
<tr><td>ec2:RevokeSecurityGroupIngress</td><td>Whitelist</td><td></td></tr>
<tr><td>ec2:RunInstances</td><td>Admin</td><td>Only Admin can create new instances.</td></tr>
<tr><td>ec2:RunScheduledInstances</td><td>Admin</td><td></td></tr>
<tr><td>ec2:StartInstances</td><td>Operator</td><td>Operators can start stop and reboot existing instances.</td></tr>
<tr><td>ec2:StopInstances</td><td>Operator</td><td>Operators can start stop and reboot existing instances.</td></tr>
<tr><td>ec2:TerminateInstances</td><td>Admin</td><td>Only Admin can terminate instances.</td></tr>
<tr><td>ec2:UnassignIpv6Addresses</td><td>Admin</td><td>Private IP addresses are within the allocated space to the account so can be safely managed by the account.</td></tr>
<tr><td>ec2:UnassignPrivateIpAddresses</td><td>Admin</td><td>Private IP addresses are within the allocated space to the account so can be safely managed by the account.</td></tr>
<tr><td>ec2:UnmonitorInstances</td><td>Admin</td><td>Monitoring frequency is managed by accounts.</td></tr>
<tr><td>ec2:UpdateSecurityGroupRuleDescriptionsEgress</td><td>Operator</td><td>Operator can update the description field only.</td></tr>
<tr><td>ec2:UpdateSecurityGroupRuleDescriptionsIngress</td><td>Operator</td><td>Operator can update the description field only.</td></tr>
<tr><td>elasticloadbalancing:AddListenerCertificates</td><td>Admin</td><td>Admin can add specified certificate to the specified secure listener.</td></tr>
<tr><td>elasticloadbalancing:AddTags</td><td>Operator</td><td>Operators can manage tags metadata about ELB.</td></tr>
<tr><td>elasticloadbalancing:ApplySecurityGroupsToLoadBalancer</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:AttachLoadBalancerToSubnets</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:ConfigureHealthCheck</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:CreateAppCookieStickinessPolicy</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:CreateLBCookieStickinessPolicy</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:CreateListener</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:CreateLoadBalancer</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:CreateLoadBalancerListeners</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:CreateLoadBalancerPolicy</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:CreateRule</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:CreateTargetGroup</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:DeleteListener</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:DeleteLoadBalancer</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:DeleteLoadBalancerListeners</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:DeleteLoadBalancerPolicy</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:DeleteRule</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:DeleteTargetGroup</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:DeregisterInstancesFromLoadBalancer</td><td>Operator</td><td>Operators can manage individual instances on the ELB as part of being able to stop start and reboot servers.</td></tr>
<tr><td>elasticloadbalancing:DeregisterTargets</td><td>Operator</td><td>Operators can manage individual instances on the ALB as part of being able to stop start and reboot servers.</td></tr>
<tr><td>elasticloadbalancing:DescribeAccountLimits</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeInstanceHealth</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeListenerCertificates</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeListeners</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeLoadBalancerAttributes</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeLoadBalancerPolicies</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeLoadBalancerPolicyTypes</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeLoadBalancers</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeRules</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeSSLPolicies</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeTags</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeTargetGroupAttributes</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeTargetGroups</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DescribeTargetHealth</td><td>Metadata</td><td></td></tr>
<tr><td>elasticloadbalancing:DetachLoadBalancerFromSubnets</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:DisableAvailabilityZonesForLoadBalancer</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:EnableAvailabilityZonesForLoadBalancer</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:ModifyListener</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:ModifyLoadBalancerAttributes</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:ModifyRule</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:ModifyTargetGroup</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:ModifyTargetGroupAttributes</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:RegisterInstancesWithLoadBalancer</td><td>Operator</td><td>Operators can manage individual instances on the ELB as part of being able to stop start and reboot servers.</td></tr>
<tr><td>elasticloadbalancing:RegisterTargets</td><td>Operator</td><td>Operators can manage individual instances on the ELB as part of being able to stop start and reboot servers.</td></tr>
<tr><td>elasticloadbalancing:RemoveListenerCertificates</td><td>Admin</td><td>Admin can remove the specified certificate from the specified secure listener.</td></tr>
<tr><td>elasticloadbalancing:RemoveTags</td><td>Operator</td><td>Operators can manage tags metadata about ELB.</td></tr>
<tr><td>elasticloadbalancing:SetIpAddressType</td><td>Admin</td><td></td></tr>
<tr><td>elasticloadbalancing:SetLoadBalancerListenerSSLCertificate</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:SetLoadBalancerPoliciesOfListener</td><td>Admin</td><td>Accounts can manage ELB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:SetRulePriorities</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:SetSecurityGroups</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>elasticloadbalancing:SetSubnets</td><td>Admin</td><td>Accounts can manage ALB configuration within cluster defined network boundaries.</td></tr>
<tr><td>iam:PassRole</td><td>Admin</td><td></td></tr>
<tr><td>marketplacecommerceanalytics:GenerateDataSet</td><td>Admin</td><td>Administrators may access product and customer data on the AWS Marketplace.</td></tr>
<tr><td>marketplacecommerceanalytics:StartSupportDataExport</td><td>Admin</td><td>Administrators may access product and customer data on the AWS Marketplace.</td></tr>
</table>
