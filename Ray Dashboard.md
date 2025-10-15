****Ray Dashboard****

While the Ray cluster initializes and loads the large language model into memory (which takes a few minutes), let's monitor the cluster's status through the Ray Dashboard. The dashboard provides valuable visibility into jobs, actors, and overall cluster health.

First, set up port forwarding to access the Ray Dashboard:

``` bash
kubectl port-forward svc/vllm 8265:8265
```

To access the dashboard, click on the Open in Browser option that appears after running the port-forward command.

accessing-ray-dashboard

Once the deployment is complete, you should see the following status indicators:

Controller status: HEALTHY
Proxy status: HEALTHY
Application status: RUNNING

This initialization process will take a few minutes to complete. During this waiting period, you can explore different sections of the dashboard or review the cluster components.

ray-dashboard

After the deployment shows as READY, let's check the status of the Ray services. Press Ctrl-c to terminate the port-forwarding, then run the following command:

``` bash

kubectl get svc


NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                        AGE
vllm            ClusterIP   172.20.208.16    <none>        6379/TCP,8265/TCP,10001/TCP,8000/TCP,8080/TCP  10m
vllm-head-svc   ClusterIP   172.20.239.237   <none>        6379/TCP,8265/TCP,10001/TCP,8000/TCP,8080/TCP  15m
vllm-serve-svc  ClusterIP   172.20.196.195   <none>        8000/TCP                                       15m

```
