RayService with vLLM on EKS

Deploying KubeRay Operator and RayService

While Ray can be deployed directly on Kubernetes using basic resources like Deployments and Services, KubeRay Operator  provides significant advantages for production environments:

1. Zero-downtime upgrades for model updates
2. High availability through external Redis for Global Control Service
3. Automated cluster lifecycle management
4. Native Kubernetes integration with custom resources
5. Simplified scaling and monitoring

In this section, we'll deploy the KubeRay Operator, which provides Kubernetes custom resource definitions (CRDs) for Ray clusters, followed by deploying a RayService configuration that sets up the serving infrastructure for our vLLM system on Amazon EKS.

For additional deployment patterns and architectures for AI workloads on EKS, refer to our AI on EKS patterns guide .

Deploy the KubeRay Operator
The KubeRay Operator manages Ray resources in Kubernetes. We'll use Helm to deploy it.

``` bash
# Add the Ray Helm repository
helm repo add kuberay https://ray-project.github.io/kuberay-helm/

# Install the KubeRay Operator
helm install kuberay-operator kuberay/kuberay-operator --version 1.1.0

# Verify the operator deployment
kubectl wait pods --for=jsonpath='{.status.phase}'=Running -l app.kubernetes.io/name=kuberay-operator --timeout=300s
```

You should see output similar to this, indicating the operator is running:

``` bash
"kuberay" has been added to your repositories
NAME: kuberay-operator
LAST DEPLOYED: Thu Sep 11 18:12:38 2025
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
pod/kuberay-operator-78cb88fb88-zlnk7 condition met
Create and Apply the RayService Configuration
```
Now we'll define our RayService configuration in a YAML file and apply it using kubectl.

```
curl -o manifests/200-inference/ray-service.yml https://raw.githubusercontent.com/aws-samples/sample-genai-on-eks/refs/tags/v1.0.0/manifests/200-ray/ray-service.yml
cat manifests/200-inference/ray-service.yml

kubectl apply -f manifests/200-inference/ray-service.yml

```

Monitor the Deployment

After deploying the RayService, we need to monitor the deployment process. It may take a few minutes for the instances to become ready and for the pods to transition to a running state.

```
kubectl get pods -w
```

Monitor Karpenter Node Scaling
Understanding the RayService Configuration
Let's break down the key components of our RayService YAML:

**Head Node Configuration
**

The head node manages the Ray cluster, orchestrating tasks like scheduling, task execution, and communication between worker nodes:

- Resources: Limited to 2 CPUs and 12GB memory, as it primarily serves as the cluster manager
- Volume Mounts: Includes paths for logs, the vLLM script, and persistent storage for models
- Node Selector: Specifically targets m5.xlarge instances for cost-effective cluster management

**Worker Node Configuration**

Worker nodes perform the actual computational tasks, particularly model inference:

- Scaling: Fixed at 1 worker (minReplicas: 1, maxReplicas: 1)
- Resources: Each worker has 28GB of memory and 1 GPU for hardware acceleration
- Volume Mounts: Same as the head node, providing access to logs, the vLLM script, and model storage
- Node Selector: Configured to run on g6e.2xlarge GPU instances optimized for inference tasks

Common Configuration Elements

Both head and worker nodes share several important configurations:

Docker Image: Both use public.ecr.aws/aws-containers/aiml/ray-2.43.0-py311-vllm0.7.3:latest, which includes:

- Ray 2.43.0 base image
- vLLM 0.7.3 capabilities
- Python 3.11 environment

Persistent Storage: Both node types mount a persistent storage volume at "/models" for storing model weights and other persistent data pulled from the MistralCDN repository 

