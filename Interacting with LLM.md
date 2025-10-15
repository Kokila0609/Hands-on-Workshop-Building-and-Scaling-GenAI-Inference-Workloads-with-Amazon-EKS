**Interacting with LLM**

This guide demonstrates how to interact with the Mistral LLM model that we deployed using vLLM in the previous section.

**Setting Up Port Forwarding**

To access the model service from your local machine, you need to set up port forwarding from the Kubernetes service to your local environment:

``` bash
kubectl port-forward svc/mistral 8080:8080
```


This command forwards traffic from your local port 8080 to port 8080 of the mistral service running in your Kubernetes cluster. Keep this terminal window open while testing the model.

Testing with a Simple Completion Request

You can test the deployed model by sending a completion request using curl. Open a new terminal window and execute the following command:

``` bash
curl -s http://localhost:8080/v1/completions \
-H "Content-Type: application/json" \
-d '{
    "model": "/ram-models/mistral-7b-v0-3",
    "prompt": "San Francisco is a",
    "max_tokens": 7,
    "temperature": 0
}' | jq

```

The response should look similar to this:

```
{
  "id": "cmpl-bbd48695ad9545ae848a17dc9a942118",
  "object": "text_completion",
  "created": 1741833663,
  "model": "/ram-models/mistral-7b-v0-3",
  "choices": [
    {
      "index": 0,
      "text": " city that is known for its beautiful",
      "logprobs": null,
      "finish_reason": "length",
      "stop_reason": null,
      "prompt_logprobs": null
    }
  ],
  "usage": {
    "prompt_tokens": 5,
    "total_tokens": 12,
    "completion_tokens": 7,
    "prompt_tokens_details": null
  }
}
```

**Conclusion**
In this session, we have:

Learned about downloading model weights to the FSx for Lustre filesystem
Deployed a Mistral LLM model using vLLM with AWS Deep Learning Containers and RAM volume optimization
Tested the deployed model by sending an inference request to the service


