
#Install MicroK8s
```
sudo apt-get update
sudo snap install microk8s --classic --channel=1.21
```
MicroK8s creates a group to enable seamless usage of commands which require admin privilege. To add your current user to the group and gain access to the .kube caching directory, run the following two commands:
```
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
```
You will also need to re-enter the session for the group update to take place:
```
su - $USER
```
Check the status
```
microk8s status --wait-ready

```
Set alias kubectl from microk8s kubectl :
```
alias kubectl='microk8s kubectl'

sudo snap install kubectl --classic
```
*******************
For more details:
https://microk8s.io/docs/getting-started
*******************
For enabling kubernetes dashboard:
https://docs.giantswarm.io/app-platform/apps/kubernetes-dashboard/
```
kubectl create serviceaccount cluster-admin-dashboard-sa
kubectl create clusterrolebinding cluster-admin-dashboard-sa \
  --clusterrole=cluster-admin \
  --serviceaccount=default:cluster-admin-dashboard-sa
  ```
  then copy the token and enter in the daswhboard.
  ```
  kubectl get secret | grep cluster-admin-dashboard-sa
  kubectl describe secret cluster-admin-dashboard-sa-token-< enter token from above code>
  ```
  
  then enable proxy:
  ```
  kubectl proxy
  ```
  *********************************
