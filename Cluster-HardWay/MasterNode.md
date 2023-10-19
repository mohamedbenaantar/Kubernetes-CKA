## Documentation link go to CHANGELOG
https://kubernetes.io/releases/

## Step1: Binary Releases (On the master node)

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

### ETCD is not part of the Binary file is a separated Project

```sh
  cd /kubernetes/server/bin/
  ###get the file and extract it
```

### Follow the documentation to tag a specific version needed
<a href="https://github.com/etcd-io/etcd/releases/tag/v3.5.9">ETCD official Documentation</a>

## Step2: Configure Certificate Authority (on the master node)

```sh
  mkdir /root/certificates
  cd /root/certificates
```
### Generate a Private key

```sh
openssl genrsa -out ca.key 2048
```

### Create a Certificate Singing Request

```sh
openssl req -new -key ca.key -subj "CN=KUBERNETES-CA" -out ca.csr
```
### Self Sign the request

```sh
  openssl x509 -req -in ca.csr -signkey ca.key -CAcreateserial -out ca.crt -days 1000
```
### Remove the csr
 
 ```sh
 rm -f ca.csr
```
### List the content of the certificate
```sh
openssl x509 -in ca.crt -text
```
