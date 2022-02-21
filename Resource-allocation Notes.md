## Setting the right requests and limits in Kubernetes:

https://learnk8s.io/setting-cpu-memory-limits-requests

************************************
To allocate the right resource:
We need 4 tools to enable:
1. metrics server (default) -- microk8s enable metrics-server
https://github.com/kubernetes-sigs/metrics-server

2. locust- opensource load testing tool for k8s
https://locust.io/

3. vertical pod autoscaler
Vertical Pod Autoscaler (VPA) frees the users from necessity of setting up-to-date resource limits and requests for the containers in their pods. 
When configured, it will set the requests automatically based on usage and thus allow proper scheduling onto nodes so that appropriate resource amount is available for each pod.
It will also maintain ratios between limits and requests that were specified in initial containers configuration.
https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler

tutorial:
https://www.kubecost.com/kubernetes-autoscaling/kubernetes-vpa/

https://www.densify.com/kubernetes-autoscaling/kubernetes-vpa

4. Goldilocks dashboard- for autoadjusting resources
https://www.fairwinds.com/blog/introducing-goldilocks-a-tool-for-recommending-resource-requests
video tutorial:
https://www.youtube.com/watch?v=WwiRDJ9THMc

another:
https://www.civo.com/learn/fairwinds-goldilocks-kubernetes-resource-recommendation-tool

another:
https://milindchawre.github.io/site/blog/fairwinds-goldilocks-kubernetes-resource-recommendation-tool/

example:
```
helm repo add fairwinds-stable https://charts.fairwinds.com/stable

helm install goldilocks fairwinds-stable/goldilocks --namespace goldilocks --set installVAP=true

kubectl get pods --namespace goldilocks
kubectl get svc

kubectl get vpa --all-namespaces

kubectl get po -n demo

kubectl label ns default goldilocks.fairwinds.com/enabled=true

kubectl get vpa -n demo

#port forward to localport 127.0.0.1:8080
kubectl -n goldilocks port-forward svc/goldilocks-dashboard 8080:80

```


****************************************
## Configure Quality of Service for Pods:
https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/

*************************************
************************************
Pipeline Resource Allocation on Kubeflow:

Pipelines are run as "workflow" kind on k8s.

set resource limits for each components of pipelines:
https://kubeflow-pipelines.readthedocs.io/en/stable/_modules/kfp/dsl/_container_op.html

