****Chat with Mistral****



**Deploy Open WebUI Application**

In this section, we'll deploy Open WebUI , a feature-rich, self-hosted web interface that serves as your gateway to interacting with Large Language Models (LLMs). This deployment will provide an intuitive and powerful interface to engage with our previously deployed Mistral-7B-Instruct-v0.3 model on EKS.

Deploy Open WebUI Application

``` bash
curl -o manifests/200-inference/openwebui.yml https://raw.githubusercontent.com/aws-samples/sample-genai-on-eks/refs/tags/v1.0.0/manifests/200-ray/openwebui.yml
cat manifests/200-inference/openwebui.yml
```

To enhance security, you can restrict access to the load balancer by allowing only your public IP address. First, determine your public IP address by visiting https://checkip.amazonaws.com/  in your web browser.

Then, update the load balancer configuration below by modifying the alb.ingress.kubernetes.io/inbound-cidrs annotation from line 80. Replace the default value 0.0.0.0/0 (which allows all IP addresses) with your-ip-address/32 (which specifies only your IP).

```
# openwebui.yml

## ...

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: open-webui-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/healthcheck-path: /
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: '10'
    alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '9'
    alb.ingress.kubernetes.io/healthy-threshold-count: '2'
    alb.ingress.kubernetes.io/unhealthy-threshold-count: '10'
    alb.ingress.kubernetes.io/success-codes: '200-302'
    alb.ingress.kubernetes.io/load-balancer-name: open-webui-ingress
    alb.ingress.kubernetes.io/inbound-cidrs: 0.0.0.0/0
...
Apply the Open WebUI deployment:


kubectl apply -f manifests/200-inference/openwebui.yml
```

Access the Application

It takes up to 2-3 minutes for the LoadBalancer to become active. Once it's ready, you can access the Open WebUI web interface through your browser.


export OPENWEBUI_URL=$(kubectl get ingress open-webui-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
aws elbv2 wait load-balancer-available --load-balancer-arns $(aws elbv2 describe-load-balancers --query 'LoadBalancers[?DNSName==`'"$OPENWEBUI_URL"'`].LoadBalancerArn' --output text)

echo "Open WebUI is ready and available at: http://${OPENWEBUI_URL}"

Click here if the URL does not show up after few seconds!

openwebui-app

Try typing different prompts in the chat interface and see how the model responds. Feel free to experiment with various questions to test the capabilities of the Mistral model!

