****vLLM Serving Script ConfigMap****

Let's deploy the ConfigMap containing our vllm_serve.py script. This ConfigMap will allow us to manage our application code separately from our Kubernetes deployment configuration, setting up the foundation for a vLLM serving system. The Python script defines a system that loads and serves large language models efficiently, handles chat completion requests in a format compatible with OpenAI's API, scales the service across multiple workers using Ray, and can be customized through environment variables.

``` bash
curl -o manifests/200-inference/vllm-serve-script.yml https://raw.githubusercontent.com/aws-samples/sample-genai-on-eks/refs/tags/v1.0.0/manifests/200-ray/vllm-serve-script.yml
cat manifests/200-inference/vllm-serve-script.yml
```
```
kubectl apply -f manifests/200-inference/vllm-serve-script.yml

Let's break down this Python code by its key components:

1. Application Setup

The code establishes a FastAPI application with necessary imports for API handling, Ray Serve integration, and vLLM engine components.
Logging is configured to track the deployment process and runtime behavior.

2. VLLMDeployment Class

Decorated with Ray Serve annotations (@serve.deployment and @serve.ingress) to make it available as a distributed service.
The constructor initializes the model with specified parameters like model path, tensor parallelism, and sequence configurations.
Creates a ModelConfig with settings optimized for inference, including data type (bfloat16) and tokenizer configuration.
3. API Endpoints

/v1/models: Returns information about available models in a format compatible with OpenAI's API.
/v1/chat/completions: Processes chat completion requests, supporting both streaming and non-streaming responses.
Follows the OpenAI API convention for easier integration with existing tools and applications.
4. Model Serving Infrastructure

Configures the vLLM engine with AsyncEngineArgs including GPU utilization settings and chunked prefill for memory optimization.
Creates the AsyncLLMEngine that handles the actual model inference operations.
Uses OpenAIServingChat to format responses according to OpenAI's expected formats.
5. Configuration and Deployment

The deployment is bound with environment variables that can be customized:
MODEL_ID: Path to the model (defaults to /models/mistral-7b-v0-3)
TENSOR_PARALLEL_SIZE: Controls how the model is split across GPUs for parallel processing
MAX_NUM_SEQS: Maximum number of sequences to process at once
MAX_MODEL_LEN: Maximum length of sequences for the model
This deployment provides a robust, scalable solution for serving LLM inferences through an OpenAI-compatible API interface, with Ray Serve handling the distribution and scaling aspects of the deployment.
