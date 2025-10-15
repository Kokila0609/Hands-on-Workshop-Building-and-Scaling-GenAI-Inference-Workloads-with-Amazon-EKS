Deploying the model using vLLM

Model and Weights

For this session, we will use the Mistral-7B-Instruct-v0.3 model . The model and weights have already been downloaded into the FSx for Lustre filesystem (to save time during the workshop). You can view the downloaded model contents by running the following command:

1
kubectl logs -l job-name=model-download

The output will look like this once the job has completed successfully:

-----
consolidated.safetensors
params.json
tokenizer.model.v3
Archive:  /tmp/mistral-tokenizer.zip (# Additional tokenizers for agents)
  inflating: config.json
  inflating: generation_config.json
  inflating: special_tokens_map.json
  inflating: tokenizer_config.json
  inflating: tokenizer.json
  inflating: tokenizer.model
The model is saved in the FSx for Lustre filesystem under the path of /models/mistral-7b-v0-3/.

consolidated.safetensors:

This is the main model weights file using the SafeTensors format
SafeTensors is a safe and efficient format for storing tensors, designed as a safer alternative to Python's pickle format
It offers better security and faster loading times compared to traditional .bin or .pt (PyTorch) files
The 'consolidated' prefix indicates that all model weights are combined into a single file
params.json:

Contains the model's configuration parameters
Includes architectural details like number of layers, attention heads, and other hyperparameters
Essential for reconstructing the model's architecture during loading
tokenizer.model.v3:

The tokenizer file that handles text preprocessing
Converts raw text into tokens that the model can understand
Contains the vocabulary and rules for text tokenization
The v3 suffix indicates the tokenizer version
Format	Key Advantages	Common Use Cases
SafeTensors (.safetensors)	- Modern, security-focused format by HuggingFace
- Zero-copy loading for better memory efficiency
- Fast loading speed and good layout control
- No file size limitations
- Supports Bfloat16 precision	- Ideal for production deployments
- Large model deployments
PyTorch (.pt/.pth)	- Native PyTorch serialization format
- Widely used in research and production
- Flexible architecture storage
- Supports multiple precision types
- Popular in academic and research environments	- Research and development
- Academic environments
- Note: Security considerations needed
TensorFlow SavedModel	- Complete model saving format for TensorFlow
- Contains model architecture and weights
- Production-ready format
- Good layout control and optimization
- Supports Bfloat16 and various precisions	- Standard TensorFlow deployments
- Production environments
HDF5 (.h5)	- Hierarchical Data Format
- Cross-platform compatibility
- Widely supported in deep learning
- Efficient for large datasets
- Popular in TensorFlow/Keras ecosystem	- Research and production
- Deep learning frameworks
- Large dataset handling
Note: This deployment uses AWS Deep Learning Containers with vLLM and includes a RAM volume optimization. An init container copies the model from FSx to a tmpfs RAM volume for faster loading, while the main vLLM container loads the model from RAM instead of directly from the network filesystem.

Deploying the Model
To deploy the model, we will use a vanilla Kubernetes service and deployment. The deployment will use AWS Deep Learning Containers with vLLM to load the model, with an optimized configuration that copies model weights to RAM for faster access.

1
2
3
4
5
mkdir -p manifests/200-inference
curl -o manifests/200-inference/vllm-deployment.yml https://raw.githubusercontent.com/aws-samples/sample-genai-on-eks/refs/heads/main/manifests/100-vllm/vllm-deployment.yml
cat manifests/200-inference/vllm-deployment.yml

kubectl apply -f manifests/200-inference/vllm-deployment.yml

Note: This deployment uses AWS Deep Learning Containers which provide optimized vLLM images with better compatibility and performance. The configuration includes an init container that copies the model from FSx to a RAM volume, significantly improving model loading times compared to direct network filesystem access.

Observing Karpenter Node Scaling
Since this deployment requires GPU nodes, Karpenter will automatically provision a new GPU-enabled node if one doesn't already exist from the previous section. You can check if GPU nodes are available:

1
2
# Check for existing GPU nodes
kubectl wait node --for=condition=Ready -l nvidia.com/gpu.present=true

If no GPU nodes exist, Karpenter will provision one automatically when the pod is scheduled. This typically takes 2-3 minutes, demonstrating Karpenter's rapid scaling capabilities for GPU workloads.

Monitoring the Deployment
First, monitor the pod status until it's running:

1
2
# Watch pod status until it transitions to Running
kubectl wait pods --for=jsonpath='{.status.phase}'=Running -l model=mistral7b --timeout=300s

Once the pod is running, you can check the logs to follow the model loading process:

1
kubectl logs -l model=mistral7b -f

You should see output similar to:

...
Defaulted container "vllm" out of: vllm, model-loader (init)
INFO 08-26 18:53:05 [launcher.py:36] Route: /v1/score, Methods: POST
INFO 08-26 18:53:05 [launcher.py:36] Route: /v1/audio/transcriptions, Methods: POST
INFO 08-26 18:53:05 [launcher.py:36] Route: /rerank, Methods: POST
INFO 08-26 18:53:05 [launcher.py:36] Route: /v1/rerank, Methods: POST
INFO 08-26 18:53:05 [launcher.py:36] Route: /v2/rerank, Methods: POST
INFO 08-26 18:53:05 [launcher.py:36] Route: /invocations, Methods: POST
INFO 08-26 18:53:05 [launcher.py:36] Route: /metrics, Methods: GET
INFO:     Started server process [10]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
The model loading process may take a few seconds to complete.

Key Configuration Parameters
In the deployment manifest above, note the important configuration parameters:

--port=8080: The port on which vLLM's inference server will listen
--model=/ram-models/mistral-7b-v0-3: Path to the model weights in RAM volume
--tokenizer_mode=auto: Using this will select the fast tokenizer if available
--trust-remote-code: Allows loading of custom model code
--gpu_memory_utilization=0.90: Percentage of GPU memory to use (90% in this case)
--max-model-len=2048: Maximum sequence length for model input
--tensor-parallel-size=1: Number of GPUs to use for tensor parallelism
--max-num-batched-tokens=8192: Maximum number of tokens that can be processed in a batch
--max-num-seqs=256: Maximum number of sequences that can be processed simultaneously
--block-size=16: Size of blocks for continuous batching
--enforce-eager: Forces eager execution mode instead of CUDA graphs
--swap-space=16: Amount of CPU memory (in GB) to use as swap space
--disable-custom-all-reduce: Disables custom all-reduce implementation for better compatibility
--config-format=mistral: Specifies Mistral-specific configuration handling for vLLM 0.9.0+ compatibility
--enable-auto-tool-choice: Enables the model to automatically select appropriate tools based on user queries
--tool-call-parser=mistral: Uses Mistral's specific format for parsing tool calls, ensuring compatibility with Mistral's function calling capabilities
