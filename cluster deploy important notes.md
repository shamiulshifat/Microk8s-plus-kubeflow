Very important serial to deploy a cluster newly or after disaster:

1. install microk8s all vms. here: https://charmed-kubeflow.io/docs/quickstart
3. add all vms to master.
4. install kubeflow from charmed. here: https://charmed-kubeflow.io/docs/quickstart
5. install velero-minio for backup.
6. take snapshot always withour kubeflow.
7. apply the snapshot.
8. deploy kuberntes dashboard.
