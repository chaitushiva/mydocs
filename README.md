Overview

EKS Auto Mode is a fully managed compute provisioning feature by AWS that simplifies running Kubernetes workloads without managing EC2 nodes, launch templates, or Karpenter setup. It leverages Karpenter under the hood but adds an abstraction layer to handle node provisioning, IAM, networking, and core add-ons.

🔧 Core Concepts

Node Classes

Define infrastructure specs: EC2 types, storage, root volume size, AMI, capacity type (On-Demand/Spot).

Automatically created during Auto Mode setup but can be modified or custom-created.

Node Pools

Logical groupings of nodes based on a Node Class.

Bind workloads to specific node pools using labels, taints, or scheduling rules.

✅ Built-in Node Pools

Node Pool

Purpose

system

Runs core system add-ons

general-purpose

Runs default application pods

These are created by default.

You can create additional node pools as needed for GPU, Spot, or other specific workloads.

Managed Instances

No launch templates or node groups needed.

Nodes are provisioned dynamically using Karpenter behind the scenes.

🎯 Features in Detail

Create Node Class

Define via console, CLI, or IaC.

Controls instance type, disk size, volume encryption, root volume type, and KMS key.

Create Node Pool

Maps workloads to Node Classes.

Labels and taints help in scoping pods.

Create Ingress Class

Auto Mode supports creating Ingress classes backed by AWS ALB.

Automatically deploys and configures AWS Load Balancer Controller.

Create Service

When exposed via type: LoadBalancer, creates AWS NLB by default.

Create StorageClass

Predefined StorageClasses include gp3.

EBS CSI driver is managed and deployed automatically.

Supports dynamic provisioning with encryption options.

Disable EKS Auto Mode

Supported via Console, AWS CLI, or IaC.

After disabling, manual management is required.

Update Kubernetes Version

EKS Auto Mode supports version upgrades.

Auto Mode ensures node compatibility and recreates incompatible nodes via its own provisioning logic (not via node groups).

Review Built-in Node Pools

system: Core add-ons.

general-purpose: Workloads.

Control Deployment

Use node selectors, affinities, and taints to control which node pool workloads run on.

Run Critical Add-ons

CoreDNS, VPC CNI, kube-proxy are deployed and upgraded automatically.

Use Network Policies

Supported with Calico or Cilium (custom add-on deployment may be required).

Tag Subnets

Public and private subnets must be tagged properly for ALB, NLB, and pod networking.

Deploy Accelerate

GPU-based Node Classes can be created to support accelerated workloads.

Generate CIS Report

Integrates with AWS Inspector and Security Hub.

CIS Benchmark Reports can be enabled.

Encrypt Root Volumes

Root volumes use EBS encryption by default.

KMS key can be overridden in Node Class config.

🧠 How It Works

Managed Instances

No Launch Templates or Node Groups involved.

EC2 lifecycle is fully managed by AWS via internal Karpenter-based provisioning engine.

Identity and Access

IAM roles for worker nodes (IRSA) are created automatically.

Add-on IRSA roles are managed by Auto Mode.

Least privilege policies applied.

Networking

Custom VPC CNI supported (advanced setup required).

Public/Private subnet configuration is respected.

Automatically handles ENI/IP assignments.

ExternalDNS controller is not automatically deployed (must deploy manually).

EFS CSI driver not deployed by default; user must install if needed.

Observability

CloudWatch Agent

Not enabled by default. Users can deploy via DaemonSet.

CloudWatch Logs/Insights

Supported via Fluent Bit add-on or third-party logging (e.g., Promtail).

Cost Considerations

You Pay For:

EKS Cluster control plane hourly fee.

EC2 Instances (On-Demand, Spot, or Savings Plan).

Additional hourly fee for using Auto Mode managed compute.

EBS volumes and associated snapshots.

Data transfer and NAT Gateway if applicable.

ALB/NLB hourly and LCU charges.

Comparison to Traditional EKS (Self-Managed + Karpenter)

Feature

EKS Auto Mode

Traditional EKS + Karpenter

EC2 Provisioning

Auto-managed (no templates)

Needs setup with CRDs

IAM for Nodes

Auto-created, managed

Manual IAM + IRSA setup

Networking

Auto ENI/IP + CNI plugin managed

Manual tagging and CNI tuning

Add-ons

Auto-managed (CoreDNS, kube-proxy, etc.)

Needs manual lifecycle management

Node Groups

❌ Not used

Optional (self-managed/MNG)

Pod Scheduling Policies

Supported via node pools + taints

Supported via Karpenter/affinities

Cluster Autoscaler

❌ Not needed

Optional; typically replaced by Karpenter

Horizontal Pod Autoscaler (HPA)

✅ Required (by user)

✅ Required (by user)

Spot + OnDemand Config

Needs separate Node Pools

Needs separate provisioners

Final Notes

EKS Auto Mode is a great fit for hands-off teams or those starting with Kubernetes.

Abstracts much of the infrastructure complexity and patching.

Not suitable for users needing fine-grained control over every piece of the infrastructure.

You can incrementally move workloads from self-managed nodes to Auto Mode node pools.

