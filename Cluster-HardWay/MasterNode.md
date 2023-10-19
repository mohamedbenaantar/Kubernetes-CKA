## Documentation link go to CHANGELOG
https://kubernetes.io/releases/

### on the master node 

```sh
  mkdir /root/binaries
  cd /root/binaries
```

```sh
  wget https://dl.k8s.io/v1.27.0/kubernetes-server-linux-amd64.tar.gz  ###<SERVER-BINARY file>
  tar -zxvf ....
```

### List the components extract it from the binary server file
```sh
  ls -l /kubernetes/server/bin/
```

###ETCD is not part of the Binary file is a separated Project

```sh
  cd /kubernetes/server/bin/
  ###get the file and extract it
```

###Follow the documentation to tag a specific version needed
<a href="https://github.com/etcd-io/etcd/releases/tag/v3.5.9">ETCD official Documentation</a>
