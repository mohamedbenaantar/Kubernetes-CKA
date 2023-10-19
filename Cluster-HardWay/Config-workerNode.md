#### Pre-Request1 : Container RunTime (WorkerNode)

  ```sh
  cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
  overlay
  br_netfilter
  EOF
  ```

  ```sh
  modprobe overlay
  modprobe br_netfilter
  ```

  ```sh
  cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
  net.bridge.bridge-nf-call-iptables  = 1
  net.ipv4.ip_forward                 = 1
  net.bridge.bridge-nf-call-ip6tables = 1
  EOF
  ```

  ```sh
  sysctl --system
  ```

  ```sh
  apt-get install -y containerd
  mkdir -p /etc/containerd
  containerd config default > /etc/containerd/config.toml
  ```
  ```sh
  nano /etc/containerd/config.toml
  ```
  --> SystemdCgroup = true

  ```sh
  systemctl restart containerd
  ```
  
  ```sh
  cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  EOF
  ```
  ```sh
  sudo sysctl --system
  ```

#### Pre-Request 2: (WORKER NODE)

  ```sh
  
  apt install -y socat conntrack ipset
  sysctl -w net.ipv4.conf.all.forwarding=1
  cd  /root/binaries/kubernetes/node/bin/
  cp kube-proxy kubectl kubelet /usr/local/bin
  ```

#### Step 1: Generate Kubelet Certificate for Worker Node (MASTER NODE)

  ```sh
  cd /root/certificates
  ```

  ```sh
  cat > kube-worker.cnf <<EOF
  [req]
  req_extensions = v3_req
  distinguished_name = req_distinguished_name
  [req_distinguished_name]
  [ v3_req ]
  basicConstraints = CA:FALSE
  keyUsage = nonRepudiation, digitalSignature, keyEncipherment
  subjectAltName = @alt_names
  [alt_names]
  DNS.1 = kplabs-cka-worker
  IP.1 = 128.199.30.177
  EOF
  ```

  ```sh
  openssl genrsa -out kube-worker.key 2048
  ```
  
  ```sh
  openssl req -new -key kube-worker.key -subj "/CN=system:node:kplabs-cka-worker/O=system:nodes" -out kube-worker.csr -config kube-worker.cnf
  openssl x509 -req -in kube-worker.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out kube-worker.crt -extensions v3_req -extfile kube-worker.cnf -days 1000
  ```

#### Step 2: Generate kube-proxy certificate (MASTER NODE)

  ```sh
  openssl genrsa -out kube-proxy.key 2048
  openssl req -new -key kube-proxy.key -subj "/CN=system:kube-proxy" -out kube-proxy.csr
  openssl x509 -req -in kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out kube-proxy.crt -days 1000
  ```

#### Step 3: Copy Certificates to Worker Node

##### - if you are using the access Key through VM (Worker Node) follow the instructions below

  ```sh
  vi /etc/ssh/sshd_config
  ---> PasswordAuthentication yes
  systemctl restart sshd
  ```
### - we need to generate a User to access the VM from the master node using scp "Create the User at Worker Node level"

  ```sh
  useradd mohamedbeno
  passwd mohamedbeno
  ```
##### - Master Node

  ```sh
  scp kube-proxy.crt kube-proxy.key kube-worker.crt kube-worker.key ca.crt mohamedbeno@IP-ADDR:/tmp
 
  ```

##### - Worker Node

  ```sh
  mkdir /root/certificates
  cd /tmp
  mv kube-proxy.crt kube-proxy.key kube-worker.crt kube-worker.key ca.crt /root/certificates
  ```

#### Step 4: Move Certificates to Specific Location. (WORKER NODE)

  ```sh
  mkdir /var/lib/kubernetes
  cd /root/certificates
  cp ca.crt /var/lib/kubernetes
  mkdir /var/lib/kubelet
  mv kube-worker.crt  kube-worker.key  kube-proxy.crt  kube-proxy.key /var/lib/kubelet/
  ```

#### Step 5: Generate Kubelet Configuration YAML File: (WORKER NODE)

  ```sh
  cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
  kind: KubeletConfiguration
  apiVersion: kubelet.config.k8s.io/v1beta1
  authentication:
    anonymous:
      enabled: false
    webhook:
      enabled: true
    x509:
        clientCAFile: "/var/lib/kubernetes/ca.crt"
  authorization:
    mode: Webhook
  clusterDomain: "cluster.local"
  clusterDNS:
    - "10.32.0.10"
  runtimeRequestTimeout: "15m"
  cgroupDriver: systemd
  EOF
  ```

#### Step 6: Generate Systemd service file for kubelet: (WORKER NODE)

  ```sh
  cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
  [Unit]
  Description=Kubernetes Kubelet
  Documentation=https://github.com/kubernetes/kubernetes
  After=containerd.service
  Requires=containerd.service
  
  [Service]
  ExecStart=/usr/local/bin/kubelet \
    --config=/var/lib/kubelet/kubelet-config.yaml \
    --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \
    --kubeconfig=/var/lib/kubelet/kubeconfig \
    --v=2
  Restart=on-failure
  RestartSec=5
  
  [Install]
  WantedBy=multi-user.target
  EOF
```

#### Step 7: Generate the Kubeconfig file for Kubelet (WORKER NODE)

  ```sh
  cd /var/lib/kubelet
  cp /var/lib/kubernetes/ca.crt .
  SERVER_IP=IP-OF-API-SERVER
  ```
```sh
{
  kubectl config set-cluster kubernetes-from-scratch \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://${SERVER_IP}:6443 \
    --kubeconfig=kube-worker.kubeconfig

  kubectl config set-credentials system:node:kplabs-cka-worker \
    --client-certificate=kube-worker.crt \
    --client-key=kube-worker.key \
    --embed-certs=true \
    --kubeconfig=kube-worker.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-from-scratch \
    --user=system:node:kplabs-cka-worker \
    --kubeconfig=kube-worker.kubeconfig

  kubectl config use-context default --kubeconfig=kube-worker.kubeconfig
}
```

  ```sh
  mv kube-worker.kubeconfig kubeconfig
  ```

### Kube-Proxy

#### Step 1: Copy Kube Proxy Certificate to Directory:  (WORKER NODE)

  ```sh
  mkdir /var/lib/kube-proxy
  ```

#### Step 2: Generate KubeConfig file

  ```sh
  {
    kubectl config set-cluster kubernetes-from-scratch \
      --certificate-authority=ca.crt \
      --embed-certs=true \
      --server=https://${SERVER_IP}:6443 \
      --kubeconfig=kube-proxy.kubeconfig
  
    kubectl config set-credentials system:kube-proxy \
      --client-certificate=kube-proxy.crt \
      --client-key=kube-proxy.key \
      --embed-certs=true \
      --kubeconfig=kube-proxy.kubeconfig
  
    kubectl config set-context default \
      --cluster=kubernetes-from-scratch \
      --user=system:kube-proxy \
      --kubeconfig=kube-proxy.kubeconfig
  
    kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
  }
  ```
  ```sh
  mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
  ```

#### Step 3: Generate kube-proxy configuration file: (WORKER NODE)

  ```sh
  cd /var/lib/kube-proxy
  ```

  ```sh
  cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
  kind: KubeProxyConfiguration
  apiVersion: kubeproxy.config.k8s.io/v1alpha1
  clientConnection:
    kubeconfig: "/var/lib/kube-proxy/kubeconfig"
  mode: "iptables"
  clusterCIDR: "10.200.0.0/16"
  EOF
  ```

#### Step 4: Create kube-proxy service file: (WORKER NODE)

  ```sh
  cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
  [Unit]
  Description=Kubernetes Kube Proxy
  Documentation=https://github.com/kubernetes/kubernetes
  
  [Service]
  ExecStart=/usr/local/bin/kube-proxy \\
    --config=/var/lib/kube-proxy/kube-proxy-config.yaml
  Restart=on-failure
  RestartSec=5
  
  [Install]
  WantedBy=multi-user.target
  EOF
  ```

#### Step 5: (WORKER NODE)
  
  ```sh
  systemctl start kubelet
  systemctl start kube-proxy
  systemctl enable kubelet
  systemctl enable kube-proxy
  ```

#### Step 6: (MASTER NODE)

```sh
kubectl get nodes
```
