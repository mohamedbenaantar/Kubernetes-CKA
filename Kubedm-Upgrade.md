### If the cluster already setUp using kubeadm follow the Instructions to Upgrade it

```sh
  apt-cache madison kubeadm
  apt-mark unhold kubelet kubectl kubeadm
```
```sh
  apt-get update && apt-get install -y kubelet=1.27.0-00 kubectl=1.27.0-00 kubeadm=1.27.0-00
```

### hold the components so cannot be removed or update it 

```sh
  apt-mark hold kubelet kubectl
```

### Upgrade it to the target version

```sh
  kubeadm upgrade plan
  kubeadm upgrade apply 1.27.0
```

## For more Info Check the documentation
<a href="https://v1-27.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/">Upgrade the Kubeadm Cluster</a>

### Update the kubelet 
```sh
systemctl daemon-reload
systemctl restart kubelet

```
