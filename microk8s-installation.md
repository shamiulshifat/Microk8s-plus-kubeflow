
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
sudo microk8s enable dashboard
microk8s kubectl describe secret -n kube-system microk8s-dashboard-token
sudo microk8s kubectl proxy &
---------
then go to this address:
  
  http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#/login
-------------------
sudo microk8s kubectl create serviceaccount cluster-admin-dashboard-sa
sudo microk8s kubectl create clusterrolebinding cluster-admin-dashboard-sa \
  --clusterrole=cluster-admin \
  --serviceaccount=default:cluster-admin-dashboard-sa
  ```
  then copy the token.
  ```
  sudo microk8s kubectl get secret | grep cluster-admin-dashboard-sa
  sudo microk8s kubectl describe secret cluster-admin-dashboard-sa-token-< enter token from above code>
  ```
  
  then enable proxy:
  ```
  sudo microk8s kubectl proxy
  ```
  
  then go to this address:
  
  http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#/login
  
  enter your copied token and save on browser for future login.
  *********************************
  To kill the proxy:
  
  https://dev4devs.com/2020/05/25/how-to-kill-the-kubectl-proxy/
  
  If above token not work:
  
  https://microk8s.io/docs/addon-dashboard

  ## for adding gpu node:
  dont use --worker
