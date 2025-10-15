Optimizing GPU Infrastructure for LLM Inference
This session will guide you through optimizing GPU infrastructure for high-performance LLM inference. You'll learn to implement dynamic GPU node provisioning with Karpenter, accelerate container startup times with SOCI snapshotter, and validate GPU functionality - creating a cost-effective and scalable foundation for production LLM workloads.

Karpenter GPU Node Provisioning
For LLM inference with GPU workloads, we use Karpenter  for automatic node provisioning. Karpenter dynamically provisions GPU nodes on-demand when workloads require them, providing cost-effective and efficient scaling.

Let's explore the Karpenter NodePool configuration:

1
2
3
4
5
# Check Karpenter NodePool configuration
kubectl get nodepool/gpu

# View Karpenter NodeClass for GPU instances
kubectl get ec2nodeclass/gpu

Our Karpenter configuration includes:


Karpenter NodePool
Instance Types: GPU-enabled instances (g6e.2xlarge)
AMI: EKS-optimized Accelerated Amazon Linux 2023 with NVIDIA support
SOCI Snapshotter: Enabled for faster container image loading
Taints: nvidia.com/gpu:NoSchedule to ensure GPU workloads only
Labels: nvidia.com/gpu.present=true for GPU node identification
SOCI Snapshotter for Faster Container Startup
The NodeClass includes SOCI (Seekable OCI) snapshotter configuration, which significantly accelerates container startup times for large images. This is particularly beneficial for ML workloads with multi-GB container images. SOCI comes pre-installed with Amazon Linux 2023 and Bottlerocket AMIs - we just need to enable it in our configuration.

1
2
# Check SOCI snapshotter configuration in the NodeClass
kubectl get ec2nodeclass gpu -o jsonpath='{.spec.userData}' | grep -B 4 -A 20 "soci"

For detailed information about SOCI implementation, see: Introducing Seekable OCI Parallel Pull mode for Amazon EKS 

GPU Node Verification
Since Karpenter provisions nodes on-demand, initially you may not see any GPU nodes:

1
2
# Check for existing GPU nodes (may be empty initially)
kubectl get nodes -l nvidia.com/gpu.present=true

NVIDIA Device Plugin
The NVIDIA device plugin runs as a DaemonSet and automatically configures GPU nodes when they're provisioned:

1
2
# Verify device plugin configuration (maybe be 0 initially since no GPU nodes are launched)
kubectl get daemonset -n nvidia-device-plugin

Adding ODCR Capacity Reservation
Before testing GPU functionality, we need to configure our Ec2NodeClass to use On-Demand Capacity Reservations (ODCR). This ensures that our GPU workloads have guaranteed capacity when needed.

First, let's get the ODCR ID for our GPU instances:

1
2
3
4
5
6
7
# Extract region from kubectl context
AWS_REGION=$(kubectl config get-contexts | grep '*' | awk '{print $2}' | cut -d':' -f4)
echo "Using AWS Region: $AWS_REGION"

# Get the ODCR ID from the current region
ODCR_ID=$(aws ec2 describe-capacity-reservations --region $AWS_REGION --filters "Name=state,Values=active" --query "CapacityReservations[0].CapacityReservationId" --output text)
echo "Found ODCR ID: $ODCR_ID"

Now, let's patch the GPU Ec2NodeClass to include the capacity reservation selector:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
# Create a patch file
mkdir -p manifests/100-introduction
cat <<EOF > manifests/100-introduction/patch-gpu-nodeclass.yaml
apiVersion: karpenter.k8s.aws/v1
kind: EC2NodeClass
metadata:
  name: gpu
spec:
  capacityReservationSelectorTerms:
    - id: $ODCR_ID
EOF

# Apply the patch
kubectl patch ec2nodeclass gpu --patch-file manifests/100-introduction/patch-gpu-nodeclass.yaml --type=merge

# Verify the patch was applied
kubectl get ec2nodeclass gpu -o yaml | grep -A 3 capacityReservationSelectorTerms

This configuration ensures that when Karpenter provisions GPU nodes, it will use instances from your ODCR, providing guaranteed capacity for your GPU workloads.

Testing GPU Functionality and Showcasing Karpenter
Let's demonstrate how Karpenter automatically provisions GPU nodes when needed by deploying a GPU workload:

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
# Deploy nvidia-smi test to trigger GPU node provisioning
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nvidia-smi
spec:
  tolerations:
  - key: "nvidia.com/gpu"
    operator: "Exists"
    effect: "NoSchedule"
  nodeSelector:
    nvidia.com/gpu.present: "true"
  containers:
  - name: nvidia-smi
    image: public.ecr.aws/amazonlinux/amazonlinux:2023-minimal
    command: ["nvidia-smi"]
    resources:
      limits:
        nvidia.com/gpu: 1
  restartPolicy: OnFailure
EOF

Now watch Karpenter provision a GPU node:

1
2
3
4
# Watch node provisioning and device plugin scaling
kubectl get nodes -w
kubectl get daemonset -n nvidia-device-plugin -w
kubectl get pod nvidia-smi -w

Note: The -w flag enables watch mode, which continuously monitors for changes. Use Ctrl + C to stop watching and proceed to the next step once you've observed the node provisioning process.

You should see:

Pod Status: Initially Pending due to no available GPU nodes
Node Creation: New GPU node appearing in kubectl get nodes
Device Plugin: DaemonSet READY count changes from 0/0 to 1/1
Pod Scheduling: Pod transitions to Running once node is ready
This typically takes 30 seconds to 1 minute for a new GPU node to become available.

Check the nvidia-smi output:

1
kubectl logs nvidia-smi

The nvidia-smi output should display detailed information about the GPU, including:

GPU model and architecture
CUDA user-mode driver version (libcuda.so)
Memory capacity and usage
Power usage and temperature
Running processes (if any)
Your cluster architecture with Karpenter and NVIDIA Device Plugin:

![overview_diagram](https://static.us-east-1.prod.workshops.aws/85fd70c2-7095-4188-af02-416500d6fec5/static/images/100-intro/gpu-overview.png?Key-Pair-Id=K36Q2WVO3JP7QD&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9zdGF0aWMudXMtZWFzdC0xLnByb2Qud29ya3Nob3BzLmF3cy84NWZkNzBjMi03MDk1LTQxODgtYWYwMi00MTY1MDBkNmZlYzUvKiIsIkNvbmRpdGlvbiI6eyJEYXRlTGVzc1RoYW4iOnsiQVdTOkVwb2NoVGltZSI6MTc2MTEwMDQ0N319fV19&Signature=IBwHtwA6vL3wtRrGIPjMmg3qg5GsLWbaQ1T81l8BNMBc6nkrxRfBhS5EsRH93TVl1ZETfUp09pKBk7KBGdndhnqh5NXrpwhFZ1woG9h1xQZY1USoQAVloR3W509oHSWiden6DgpSoX13iRQnnihQ7JaAnS5Nv%7EVg9cN1WA2Icrm4K7-gc9frZ%7EmQH3R7%7Ek1TbiA4D6WwfP4-%7EVe2paNHWrOrA6xbvy%7ECbbhvzhJtDtVkNrjD8daPoYw97fSUq0tkSx7KYPV7PCj-6hcoj5Wt4TrjqNg8hmJkTq9X7wM1nOucDt0is%7EG11UsPQ9iSVbKBhSbLuexVSz-cNjmgT7zl4g__)

