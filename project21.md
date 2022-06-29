# K8s From Grounds Up
<img width="1010" alt="Screenshot 2022-06-27 at 13 02 07" src="https://user-images.githubusercontent.com/33035619/175936654-d082af95-2d33-4dae-9349-a59f7ec21690.png">


## Install Client Tools
Tools:
- awscli: Tool used to manage AWS resources
  ```
  Installation: https://aws.amazon.com/cli/
  ```
- kubectl: Used to manage K8s cluster
  ```
  wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl
  chmod +x kubectl
  sudo mv kubectl /usr/local/bin/
  kubectl version --client
  ```
- cfssl: an open source toolkit for everything TLS/SSL from Cloudflare
- cfssljson – a program, which takes the JSON output from the cfssl and writes certificates, keys, CSRs, and bundles to disk.
  ```
  wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
  chmod +x cfssl cfssljson
  sudo mv cfssl cfssljson /usr/local/bin/
  ```

## AWS Resources for K8s Cluster
- VPC (172.31.0.0/16 CIDR block)
  - Enable DNS support
  - Enable DNS hostnames
  ![image](https://user-images.githubusercontent.com/50557587/162780073-79ff491b-7c30-4981-986f-2968522c032a.png)
  
  - DHCP option sets
  ![image](https://user-images.githubusercontent.com/50557587/162781280-ec1f6cb7-6f3a-417c-b4af-d422947cdfa5.png)
  
  ![image](https://user-images.githubusercontent.com/50557587/162781512-d83d0333-8429-4a46-aa6c-96ee384e4b89.png)
  
  - Subnet (172.31.0.0/24 CIDR block)
  
  ![image](https://user-images.githubusercontent.com/50557587/162781884-3fd33496-57bd-4c92-bdfd-1b6320a473e4.png)
  
  - Internet Gateway
  ![image](https://user-images.githubusercontent.com/50557587/162782263-e6882fe7-9776-44f3-b6ed-a8e49040be13.png)
  
  - Route table with route to the internet gateway
  
  ![image](https://user-images.githubusercontent.com/50557587/162782515-698a8141-b467-4a1a-927c-371ff32c7c9a.png)
  
  ![image](https://user-images.githubusercontent.com/50557587/162782551-1dc3ddbf-0501-49e8-b0cb-cdb4eac05e58.png)
  - Security groups
    - Allow ports 2379-2380 from within the subnet
    - Allow ports 30000-32767 from anywhere (to access cluster services)
    - Allow port 6443 for K8s API server
    - Allow SSH connection from anywhere
    - Allow ICMP Rule
  ![image](https://user-images.githubusercontent.com/50557587/162782985-25e6662c-a264-4015-a36a-14874b4b721c.png)
  
- Network load balancer
![image](https://user-images.githubusercontent.com/50557587/162783334-4c2c31a5-2862-4a18-a160-d7c68bf49fda.png)
  - Target group
    - Protocol: TCP
    - Port: 6443

  - ELB listener
    - Protocol: TCP
    - Port: 6443
    - Default actions: forward to target group
    
![image](https://user-images.githubusercontent.com/50557587/162784238-d80f05c9-e672-464d-8e59-1faed20fab09.png)

![image](https://user-images.githubusercontent.com/50557587/162784363-6cfa4a55-e7f2-44d7-98f5-95af98caac7c.png)

![image](https://user-images.githubusercontent.com/50557587/162784442-57a4e8a4-17ab-4006-9b0a-293c39b62ee2.png)

![image](https://user-images.githubusercontent.com/50557587/162784676-ae98744d-3683-4f9e-aad2-a968206d48e2.png)

- Create 3 master nodes and 3 worker nodes (disable source-dest checks)
  
## Prepare Self Signed CA and generate TLS Certificates
- Create ca-authority directory
- Generate the CA configuration file, root certificate and private key
- Generate the Certificate Signing Request (CSR), Private Key and the Certificate for the Kubernetes Master Nodes
- kube-scheduler Client Certificate and Private Key
- kube-controller-manager Client Certificate and Private Key
- kube-proxy Client Certificate and Private Key
- kubelet Client Certificate and Private Key
- Admin user Certificate and Private Key
- Service account certificate and private key

![image](https://user-images.githubusercontent.com/50557587/162786848-9f5e8e94-5f3d-423e-86ce-e3ab5b23e496.png)

![image](https://user-images.githubusercontent.com/50557587/162788067-1d03d47f-300b-4d43-b995-62b8503929fe.png)

![image](https://user-images.githubusercontent.com/50557587/162788196-0c66dabf-1af1-4c6b-bb31-1e95e1250072.png)

![image](https://user-images.githubusercontent.com/50557587/162788739-079719ee-922b-45ee-ae5c-c21541ee662a.png)

![image](https://user-images.githubusercontent.com/50557587/164046891-d8ab979a-4966-450c-99c3-45891060c7b2.png)


```
for i in 0 1 2; do
  instance="${NAME}-worker-${i}"
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/${NAME}.id_rsa \
    kube-proxy.kubeconfig k8s-cluster-from-ground-up-worker-${i}.kubeconfig ubuntu@${external_ip}:~/; \
done
```

```
for i in 0 1 2; do
  instance="${NAME}-master-${i}"
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/${NAME}.id_rsa \
    admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig  ubuntu@${external_ip}:~/; \
done
```

![image](https://user-images.githubusercontent.com/50557587/164048787-3cd1eba5-54b4-48a9-abe4-254b1b78929e.png)
  
## Distribute the client and server certificates
- Copy specific worker node keys to the respective worker nodes
- Copy kube-proxy, kubelet and ca.pem to all worker nodes
- Copy ca.pem, ca-key.pem, service-account* and master-kubernetes* to all master nodes

## Generate K8s configuration files with kubectl
- Generate configuration files for individual worker instances
- Generate the kube-proxy kubeconfig
- Generate the kube-controller-manager kubeconfig
- Generate the kube-scheduler kubeconfig
- Generate the kubeconfig file for cluster administrator
- Copy kube-controller and kube-scheduler files to all master nodes, kube-proxy and individual worker node kubeconfig files to the worker nodes
  
## Prepare ETCD database for encryption at rest
- ETCD encryption should be 32 bytes (head -c 32 /dev/urandom | base64)
- Create encyption-config.yaml file
- Bootstrap etcd cluster
<img width="979" alt="Screenshot 2022-06-27 at 13 03 11" src="https://user-images.githubusercontent.com/33035619/175937971-77b6dbd9-fec8-467d-83ba-c11df470b808.png">

## Bootstrap controlplane
- Create /etc/kubernetes/config directory
- Download and install kubectl, kube-apiserver, kube-scheduler and kube-controller-manager
- Configure each of the services
<img width="978" alt="Screenshot 2022-06-27 at 13 03 20" src="https://user-images.githubusercontent.com/33035619/175938157-e77b9827-7fc1-4e6b-8469-c3ac95640202.png">
<img width="974" alt="Screenshot 2022-06-27 at 13 03 35" src="https://user-images.githubusercontent.com/33035619/175938168-92d87f0b-a208-40b1-89e1-34c0abe755fa.png">
<img width="975" alt="Screenshot 2022-06-27 at 13 03 41" src="https://user-images.githubusercontent.com/33035619/175938176-7e79bb5a-2630-411f-957b-98c6b41d63de.png">

## Test current setup
```
kubectl cluster-info --kubeconfig admin.kubeconfig
```

<img width="1008" alt="Screenshot 2022-06-27 at 13 03 51" src="https://user-images.githubusercontent.com/33035619/175938294-ca9a18e5-263e-448b-b233-12eb654ae053.png">


```
kubectl get namespaces --kubeconfig admin.kubeconfig
```
<img width="772" alt="Screenshot 2022-06-27 at 13 03 58" src="https://user-images.githubusercontent.com/33035619/175938397-da6b1172-a6ae-4489-9559-baf6524cf41e.png">
```
curl --cacert /var/lib/kubernetes/ca.pem https://$INTERNAL_IP:6443/version
```
<img width="983" alt="Screenshot 2022-06-27 at 13 04 05" src="https://user-images.githubusercontent.com/33035619/175938406-17aeadeb-514f-479c-b721-6fc5cb9d13f2.png">
<img width="1012" alt="Screenshot 2022-06-27 at 13 04 16" src="https://user-images.githubusercontent.com/33035619/175938412-4e96e3ae-fbb3-478c-81b6-cf51de904a00.png">

## Configuring the worker nodes
- Create and apply cluster role binding for kubelet
  ```
  cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
    annotations:
        rbac.authorization.kubernetes.io/autoupdate: "true"
    labels:
        kubernetes.io/bootstrapping: rbac-defaults
    name: system:kube-apiserver-to-kubelet
    rules:
    - apiGroups:
        - ""
        resources:
        - nodes/proxy
        - nodes/stats
        - nodes/log
        - nodes/spec
        - nodes/metrics
        verbs:
        - "*"
    EOF
  ```
  ```
  cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
    name: system:kube-apiserver
    namespace: ""
    roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: system:kube-apiserver-to-kubelet
    subjects:
    - apiGroup: rbac.authorization.k8s.io
        kind: User
        name: kubernetes
    EOF
  ```
<img width="795" alt="Screenshot 2022-06-27 at 13 14 44" src="https://user-images.githubusercontent.com/33035619/175938789-35c76963-51d3-4e2e-8e74-1d15bf6c16ef.png">

<img width="917" alt="Screenshot 2022-06-27 at 13 15 38" src="https://user-images.githubusercontent.com/33035619/175938950-ec90f44a-9c8d-434c-8249-b8f42171eb24.png">

- Install OS dependencies
  ```
  {
  sudo apt-get update
  sudo apt-get -y install socat conntrack ipset
  }
  ```
- Disable swap
  ```
  sudo swapoff -a
  ```
- Download binaries for runc, crictl and containerd
  ```
   wget https://github.com/opencontainers/runc/releases/download/v1.0.0-rc93/runc.amd64 \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.21.0/crictl-v1.21.0-linux-amd64.tar.gz \
  https://github.com/containerd/containerd/releases/download/v1.4.4/containerd-1.4.4-linux-amd64.tar.gz 
  ```
- Configure containerd
  ```
  {
  mkdir containerd
  tar -xvf crictl-v1.21.0-linux-amd64.tar.gz
  tar -xvf containerd-1.4.4-linux-amd64.tar.gz -C containerd
  sudo mv runc.amd64 runc
  chmod +x  crictl runc  
  sudo mv crictl runc /usr/local/bin/
  sudo mv containerd/bin/* /bin/
  }
  ```
  `sudo mkdir -p /etc/containerd/`
  ```ini
  cat << EOF | sudo tee /etc/containerd/config.toml
    [plugins]
    [plugins.cri.containerd]
        snapshotter = "overlayfs"
        [plugins.cri.containerd.default_runtime]
        runtime_type = "io.containerd.runtime.v1.linux"
        runtime_engine = "/usr/local/bin/runc"
        runtime_root = ""
    EOF
  ```
  Create systemd service file for containerd
  ```ini
  cat <<EOF | sudo tee /etc/systemd/system/containerd.service
    [Unit]
    Description=containerd container runtime
    Documentation=https://containerd.io
    After=network.target

    [Service]
    ExecStartPre=/sbin/modprobe overlay
    ExecStart=/bin/containerd
    Restart=always
    RestartSec=5
    Delegate=yes
    KillMode=process
    OOMScoreAdjust=-999
    LimitNOFILE=1048576
    LimitNPROC=infinity
    LimitCORE=infinity

    [Install]
    WantedBy=multi-user.target
    EOF
  ```
  Create directories for to configure kubelet, kube-proxy, cni, and a directory to keep the kubernetes root ca file:
  ```
    sudo mkdir -p \
    /var/lib/kubelet \
    /var/lib/kube-proxy \
    /etc/cni/net.d \
    /opt/cni/bin \
    /var/lib/kubernetes \
    /var/run/kubernetes
  ```

- Download and install CNI
  ```
  wget -q --show-progress --https-only --timestamping \
  https://github.com/containernetworking/plugins/releases/download/v0.9.1/cni-plugins-linux-amd64-v0.9.1.tgz
  ```
  ```
  sudo tar -xvf cni-plugins-linux-amd64-v0.9.1.tgz -C /opt/cni/bin/
  ```
- Download binaries for kubelet, kubectl and kube-proxy
  ```
  wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubelet
  ```
  Install binaries
  ```
  {
  chmod +x  kubectl kube-proxy kubelet  
  sudo mv  kubectl kube-proxy kubelet /usr/local/bin/
  }
  ```

## Configure Worker node components
- Configure bridge and loopback networks
  ```
  cat > 172-20-bridge.conf <<EOF
    {
        "cniVersion": "0.3.1",
        "name": "bridge",
        "type": "bridge",
        "bridge": "cnio0",
        "isGateway": true,
        "ipMasq": true,
        "ipam": {
            "type": "host-local",
            "ranges": [
            [{"subnet": "${POD_CIDR}"}]
            ],
            "routes": [{"dst": "0.0.0.0/0"}]
        }
    }
    EOF
  ```
  Loopback:
  ```
  cat > 99-loopback.conf <<EOF
    {
        "cniVersion": "0.3.1",
        "type": "loopback"
    }
    EOF
  ```

- Move files to network configuration directory
  ```
  sudo mv 172-20-bridge.conf 99-loopback.conf /etc/cni/net.d/
  ```

- Move the certificates and kubeconfig file to their respective configuration directories
- Create the kubelet-config.yaml file
- Configure kubelet and kube-proxy systemd service
  - Kubelet service
<img width="949" alt="Screenshot 2022-06-27 at 13 17 02" src="https://user-images.githubusercontent.com/33035619/175939241-8e60c1d2-5288-44e5-b109-e9b766e489be.png">
  - Kube-proxy service
<img width="942" alt="Screenshot 2022-06-27 at 13 17 11" src="https://user-images.githubusercontent.com/33035619/175939246-f58e0018-4691-493e-b717-71649dc9303d.png">
  - `kubectl get nodes`
<img width="941" alt="Screenshot 2022-06-27 at 13 17 18" src="https://user-images.githubusercontent.com/33035619/175939263-6484e247-00fb-4584-bdef-f1bf8a5df02c.png">
