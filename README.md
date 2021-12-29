# Kubernetes-install

> Ubuntu: 18.04 <br/>Kubernetes: 1.9.0 <br/> docker: 18.06.2

### 설치 순서
#### 1. docker install <br/> 2. docker setup<br/>3. Kubernetes install<br/>4. API Server 주소<br/>5. Master 노드 생성 및 설정<br/>6. Master 노드 Flannel 설치 및 확인<br/>7. Worker 노드 설정 <br/><br/> etc


------------
#### docker install
```
sudo apt update
sudo apt-get update

sudo apt-get remove docker docker-engine docker.io
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update

sudo apt-get install docker-ce=18.06.2~ce~3-0~ubuntu -y # docker: 18.06.2 설치
```

#### docker setup
```ubuntu
$ sudo cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo mkdir -p /etc/systemd/system/docker.service.d

sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### Kubernetes install
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - && \
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
  sudo apt-get update -q && \
  sudo apt-get install -qy kubelet=1.19.0-00 kubectl=1.19.0-00 kubeadm=1.19.0-00 # Kubernetes: 1.19.0 설치
```

#### API Server 주소
  ![image](https://user-images.githubusercontent.com/37894081/147641971-4bd11ecb-eed1-4400-b62a-78a535a24fdf.png)
#### Master 노드 생성 및 설정
```
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.99.102

## Worker 노드 설정 시 필요 / Worker 노드에서 실행
'kubeadm join <ip:6443> --token <token>\
  --discoery-token-ca-cert-hash <token-hash>'

# Master 노드 생성 후 실행
mkdir -p $HOME/.kube 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Master 노드 Flannel 설치 및 확인
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# Flannel 확인
kubectl get pod --namespace=kube-system -o wide
ifconfig
```

#### Worker 노드 설정
```
# Worker 노드에서 실행!!!<중요>
kubeadm join <ip:6443> --token <token>\
  --discoery-token-ca-cert-hash <token-hash>
```

#### etc
##### Kubernetes 버전 확인
```
curl -s https://packages.cloud.google.com/apt/dists/kubernetes-xenial/main/binary-amd64/Packages | grep Version | awk '{print $2}'
```
##### 읽기전용파일 에러
```
sudo mount -o remount,rw /
```

##### docker 제거
```
sudo apt remove docker* containerd*
sudo rm -rf /var/lib/docker
sudo rm -rf /etc/docker/
rm -rf ~/.docker/
```

##### Kubernetes 초기화 및 제거
```
sudo kubeadm reset
sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*
sudo apt-get autoremove  
sudo rm -rf ~/.kube
```

##### Kubernetes 애드온(Flannel, cni) 제거
```
kubeadm reset
systemctl stop kubelet
systemctl stop docker
rm -rf /var/lib/cni/
rm -rf /var/lib/kubelet/*
rm -rf /etc/cni/
ifconfig cni0 down
ifconfig flannel.1 down
ifconfig docker0 down
```

#### Ref
> https://medium.com/finda-tech/overview-8d169b2a54ff<br/>
> https://velog.io/@dry8r3ad/Kubernetes-Cluster-Installation<br/>
> https://ssup2.github.io/record/Kubernetes_%EC%84%A4%EC%B9%98_kubeadm_Ubuntu_18.04/
