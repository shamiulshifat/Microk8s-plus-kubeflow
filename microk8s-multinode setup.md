 ## To setup a multinode cluster on microk8s:
 https://microk8s.io/docs/clustering
 
 ## Add roles to nodes:
 ![image](https://user-images.githubusercontent.com/36572440/154898571-f2f58d27-6696-48a2-83cf-855101ddbb5d.png)
 ```
kubectl label node <node name> node-role.kubernetes.io/<role name>=<key - (any name)>
```
e.g:
```
kubectl label nodes ubuntu-vm-one-1 kubernetes.io/role=worker1
```
to overwrite labels:
```
kubectl label --overwrite nodes <your_node> kubernetes.io/role=<your_new_label>


e.g.

kubectl label --overwrite nodes slave-node kubernetes.io/role=worker1
```
