****Cleanup****

**Cleanup Resources**

After completing this module, let's clean up the resources we created to avoid unnecessary costs and resource consumption. The following command will remove the deployed Mistral model and the associated service:

```bash
kubectl delete -f manifests/200-inference/vllm-deployment.yml
```

This command deletes all the Kubernetes resources defined in the YAML file, including:

1. The Mistral service
2. The Mistral deployment
3. Any associated pods running the vLLM container
4. Once completed, your cluster will no longer be running the Mistral model inference service.



