### Note: Step 1 to 3 will be on (Worker node)

### Step 1: Download CNI Plugins

  ```sh
  cd /tmp

  wget https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz

  ```
#### Step 2: Configure Base Directories

  ```sh
  mkdir -p \
    /etc/cni/net.d \
    /opt/cni/bin \
    /var/run/kubernetes
  ```
#### Step 3: Move CNI Tar File And Extract it
  
  ```sh
  mv cni-plugins-linux-amd64-v1.1.1.tgz /opt/cni/bin
  cd /opt/cni/bin
  tar -xzvf cni-plugins-linux-amd64-v1.1.1.tgz
  ```
#### Step 4: Configuring Weave (Run it on Master Node)

   ```sh
  kubectl apply -f https://raw.githubusercontent.com/zealvora/certified-kubernetes-administrator/master/Domain%206%20-%20Cluster%20Architecture%2C%20Installation%20%26%20Configuration/weave-daemonset-k8s.yaml
    ```

---> REMEMBER, you might need to restart the kubelet on the worker Node

