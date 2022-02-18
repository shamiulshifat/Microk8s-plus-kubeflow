
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
```
*******************
For more details:
https://microk8s.io/docs/getting-started
*******************
