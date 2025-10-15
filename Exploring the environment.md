Exploring the environment
In this section, we'll explore the base workshop environment that has been pre-provisioned for running generative AI workloads. Our infrastructure has been designed to support large language models (LLMs) and other AI/ML applications with the necessary compute, networking, and storage resources.

Workshop Infrastructure
We have pre-provisioned an environment consisting of:

Amazon VPC: A dedicated virtual network with public and private subnets across multiple Availability Zones.

Amazon EKS: A managed Kubernetes cluster.

EKS Managed NodeGroups: Automated node provisioning and lifecycle management for our base Kubernetes workloads.

Karpenter: Dynamic node autoscaler for on-demand GPU provisioning for AI/ML workloads.

EKS Add-Ons: A suite of essential components that we'll explore in detail throughout this section.

Overview Diagram

Connect to Your EKS Cluster
We've pre-provisioned an Amazon EKS cluster named genai-workshop for this workshop. Run these commands in the terminal to connect to your cluster:

1
2
3
4
5
# Configure kubectl for your EKS cluster [[1]](https://docs.aws.amazon.com/eks/latest/userguide/connector-grant-access.html)
aws eks update-kubeconfig --name genai-workshop

# Verify your cluster connection.
kubectl get pods --all-namespaces

Base Cluster Configuration
Our cluster starts with a basic configuration to support the workshop's control plane and essential services. Let's explore the initial setup:

To view the base nodes of our cluster:

1
kubectl get nodes -l node.kubernetes.io/instance-type=m5.xlarge

The base cluster uses:


General Purpose NodeGroup
Standard EKS-optimized Amazon Linux 2023 (AL2023) AMI
Instance type: m5.xlarge
2 nodes configured for running cluster management and non-accelerated workloads
Note
Additional compute resources for AI/ML workloads will be configured in later sections based on your chosen acceleration path (GPU).
EKS Add-ons Overview
Our EKS cluster comes configured with several add-ons to provide core functionality and workshop-specific features. Let's examine what's currently running on the cluster and why.

AWS FSX CSI Driver
The Amazon FSx for Lustre  integrates with EKS to provide high-performance file storage, which is crucial for GenAI workloads. Its CSI Driver  implements the Container Storage Interface (CSI) specification to manage these filesystems in our Kubernetes cluster.

The FSx CSI Driver consists of two main components:

Controller Pods (fsx-csi-controller-*): Handle volume management operations like provisioning and deleting FSx for Lustre file systems. These typically run as a deployment with multiple replicas for high availability.

Node Pods (fsx-csi-node-*): Run on every node in the cluster and handle mounting operations, making the FSx volumes available to pods running on that node.

To check the status of FSx CSI Driver components, run:

1
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-fsx-csi-driver

Karpenter
Karpenter  is a Kubernetes cluster autoscaler that automatically provisions right-sized compute resources in response to changing application load. For this workshop, Karpenter manages GPU node provisioning on-demand, providing cost-effective scaling for AI/ML workloads.

To check Karpenter status:

1
kubectl get pods -n karpenter

Default Cluster Add-ons
The following add-ons provide essential cluster functionality:

CoreDNS: Handles cluster DNS resolution
kube-proxy: Manages pod networking and load balancing
VPC CNI: Provides pod networking using AWS VPC IP addresses
EKS Pod Identity Agent: Enables credential management for your applications, similar to how Amazon EC2 instance profiles provide credentials to EC2 instances
Kube Prometheus Stack: A collection of Kubernetes manifests, Grafana dashboards, and Prometheus rules that provides cluster monitoring capabilities through Prometheus, a time-series database for metrics collection and alerting
Grafana Operator: A Kubernetes operator that manages Grafana instances, dashboards, and data sources, enabling automated deployment and configuration of Grafana visualization tools within the cluster
Next Steps
You now have a good understanding of our base infrastructure. As we progress through the workshop, we'll introduce additional components and add-ons when needed, making it easier to understand their specific roles in our generative AI stack. You can proceed to the next module where we'll start working with our first AI workload.

