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

4. Goldilocks dashboard- for autoadjusting resources
https://www.fairwinds.com/blog/introducing-goldilocks-a-tool-for-recommending-resource-requests


****************************************
## Configure Quality of Service for Pods:
https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/
