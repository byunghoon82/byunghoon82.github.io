---
title: "kuberntes 설치 방법"
date: 2019-09-11
tags: Kubernetes
---


## 테스트환경:
- Docker CE 18.09   
- CentOS 7
-  kubeadm version
```json
kubeadm version: &version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.3", GitCommit:"2d3c76f9091b6bec110a5e63777c332469e0cba2", GitTreeState:"clean", BuildDate:"2019-08-19T11:11:18Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}
```
- kubectl version
```json
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.3", GitCommit:"2d3c76f9091b6bec110a5e63777c332469e0cba2", GitTreeState:"clean", BuildDate:"2019-08-19T11:13:54Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.3", GitCommit:"2d3c76f9091b6bec110a5e63777c332469e0cba2", GitTreeState:"clean", BuildDate:"2019-08-19T11:05:50Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"linux/amd64"}
```


## 설치방법:
CentOS 7 설정:
```bash
yum -y update

# swap off
swapoff -a
echo 0 > /proc/sys/vm/swappiness
sed -e '/swap/ s/^#*/#/' -i /etc/fstab

# 방화벽 해제
systemctl disable firewalld
systemctl stop firewalld

# br_netfilter 모듈 추가
modprobe br_netfilter
lsmod | grep br_netfilter

# iptables 설정
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --syste

# selinux off
setenforce 0
vi /etc/selinux/config
  # disable 설정
getenforce

reboot
```

Docker 설치:
```bash
# docker가 설치되어 있다면 삭제
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine

# docker repo 추가
yum install -y yum-utils  device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 설치 가능항 docker 버전 리스트 확인
yum list docker-ce --showduplicates | sort -r

# 버전 확인 후 설치
# 2019.8.30일 기준 kubernetes는 Docker 18.09 버전까지 지원
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io

systemctl start docker
systemctl enable docker
```

kubernetes 설치:
```bash
# mater & worker 노드 공통

# kubernetes repo 추가 및 설치
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable kubelet && systemctl start kubelet
```

mater만 실행:
```bash
# flannel plugin 설치
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 초기화 with --pod-network-cidr for flannel plugin
kubeadm init --pod-network-cidr 10.244.0.0/16

# 일반 사용자가 kubectl을 사용할 수 있도록 설정
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# kubelet 시작
systemctl start kubelet
systemctl enable kubelet

# /etc/hosts 에 다음 추가
cat /etc/hosts
127.0.0.1   master localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.4 master
192.168.0.2 node1
192.168.0.3 node2
```

node1&2에서 실행:
```bash   
# master kubeam init를 통해 확인된 마지막 join 실행
kubeadm join 192.168.0.2:6443 --token <token_value> --discovery-token-ca-cert-hash sha256:<sha256_value> --ignore-preflight-errors=All

# 일반 사용자가 kubectl을 사용할 수 있도록 설정
mkdir -p $HOME/.kube
  # admin.conf는 worker노드에 생성이 되어 있지 않아 master에서 실행
  scp /etc/kubernetes/admin.conf root@192.168.0.3:/root/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# kubelet 시작
systemctl start kubelet
systemctl enable kubelet
```

쿠버네티스 동작 확인:
```bash
kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-0               Healthy   {"health": "true"}
```

join 상태 확인:
```bash
kubectl get nodes
NAME     STATUS     ROLES    AGE   VERSION
master   NotReady   master   15h   v1.15.3
node1    NotReady   <none>   15h   v1.15.3
node2    NotReady   <none>   19s   v1.15.3
```


## kubernetes 명령어:
Join 방법:
```bash
kubeadm join 192.168.0.2:6443 --token <token_value> --discovery-token-ca-cert-hash sha256:<token_value> --ignore-preflight-errors=All

# Token 생성(확인)
kubeadm token list
kubeadm token create

# Hash 확인
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

쿠버네티스의 구성요소가 Pod로 어떤 노드에 설치되어 있는지 master에서 확인:
```bash
kubectl get po -o custom-columns=POD:metadata.name,NODE:spec.nodeName --sort-by spec.nodeName -n kube-system
```

기타 명령어
```bash
kubectl get events
kubectl get pods
kubectl get deplyments
kubectl get services
kubectl describe pod <pod_name>
kubectl describe nodes <node_name>
kubectl get pod --all-namespaces
kubectl describe pod --all-namespaces
kubectl logs <pod_name>
kubectl delete namespace <namespace_name>
kubectl delete -f <yml_file_name>.yml
kubectl exec -it <pod_name> sh
kubectl kubectl describe services <service_name>
```

## Issue Track:
### node에서 master에 연결하기 위한 kubeadm join 을 실행할때 다음 에러가 확인된다.
메세지:  
```bash
[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
```

해결방법:   
  + <https://blog.naver.com/weplayicecream/221498635944>


### systemctl start kubelet 및 #systemctl status kubelet을 실행 후 하기와 같은 시작 실패 메세지가 확인되는 경우
메세지:  
```bash
systemctl status kubelet
/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS

tail /var/log/message 
F0830 14:09:16.330525   22600 server.go:198] failed to load Kubelet config file /var/lib/kubelet/config.yaml, error failed to read kubelet config file "/var/lib/kubelet/config.yaml", error: open /var/lib/kubelet/config.yaml: no such file or directory
```

해결방법:  
+ 마스터 노드: # kubeadm init 후 #systemctl start kubelet을 실행
+ 워커 노드: 마스터 노드에 # kubeadm join 후 #systemctl start kubelet을 실행


### node에서 master에 join을 할때 확인되는 메세지
메세지:  
```bash
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR DirAvailable--etc-kubernetes-manifests]: /etc/kubernetes/manifests is not empty
        [ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
        [ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
```

해결방법:  
```bash
join 명령어에 --ignore-preflight-errors=All를 추가한다.
https://github.com/kubernetes/kubeadm/issues/974
```

### join 할때 hang이 발생되었고 --v=2 추가로 원인 확인
메세지:  
```bash
[preflight] Running pre-flight checks
```

디버그방법:  
```bash
kubeadm join 121.166.116.23:6443 --token <token_value>  --discovery-token-ca-cert-hash sha256:<sha256_value> --v=2
```

메세지:  
```bash
I0830 14:40:30.903964    1707 token.go:199] [discovery] Trying to connect to API Server "121.166.111.11:6443"
I0830 14:40:30.904680    1707 token.go:74] [discovery] Created cluster-info discovery client, requesting info from "https://121.166.111.11:6443"
I0830 14:40:30.933320    1707 token.go:140] [discovery] Requesting info from "https://121.166.111.11:6443" again to validate TLS against the pinned public key
I0830 14:40:30.950202    1707 token.go:146] [discovery] Failed to request cluster info, will try again: [Get https://121.166.116.23:6443/api/v1/namespaces/kube-public/configmaps/cluster-info: x509: certificate is valid for 10.96.0.1, 192.168.0.39, not 121.166.116.23]
```

해결방법:  
+ mater: 공인:121.166.111.11 / Private: 192.168.0.39
+ node: 다른 네트워크 / 이 경우 master는 192.168.0.x 대역으로 운영되며 해당 대역에 대하여 인증서가 유효하나 같은 네트워크 망이 아닌 다른 네트워크 망에서 공인IP로 접속을 시도하여 발생되는 문제
같은 망에서 접속이 필요함


### systemctl status kubelet 으로 상태 학인시
메세지:  
```bash
Sep 04 11:47:30 master kubelet[8909]: E0904 11:47:30.202073    8909 kubelet.go:2248] node "master" not found
```

해결방법:    
master를 /etc/hosts에 추가  
```bash
# cat /hosts
127.0.0.1   master localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.2 master
192.168.0.3 node1
192.168.0.4 node1
```

### kubectl get nodes 실행 시 메세지 확인
메세지:  
```bash
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

해결방법:  
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

systemctl stop kubelet
systemctl start kubelet
```

### systemctl status kubelet 실행 시 메세지 확인
메세지:  
```bash
Sep 06 02:50:28 node1 kubelet[2843]: W0906 02:50:28.954189    2843 cni.go:213] Unable to update cni config: No networks found in /etc/cni/net.d
Sep 06 02:50:29 node1 kubelet[2843]: E0906 02:50:29.842812    2843 kubelet.go:2169] Container runtime network not ready: NetworkReady=false 
```

해결방법:  
```bash
# https://github.com/kubernetes/kubeadm/issues/1031
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### flannel을 통한 pod간 통신
문서:  
+ <https://crystalcube.co.kr/198>  
+ <https://likefree.tistory.com/17>

### kubectl describe pod <pod_name> 실행 시 메세지 확인
메세지:  
```bash
Warning  FailedCreatePodSandBox  36m                 kubelet, node2     Failed create pod sandbox: rpc error: code = Unknown desc = failed to set up sandbox container "2eb830c0c80e5077b9a6409a4f54b7c32524c58bc7cdbe9780b58736bac7c1fc" network for pod "nginx-55bc597fbf-kq5q6": NetworkPlugin cni failed to set up pod "nginx-55bc597fbf-kq5q6_default" network: open /run/flannel/subnet.env: no such file or directory
```

원인:  
master 노드에 flannel 플러그인 설치 후 pod간 통신에 필요한 네트워크를 kubeadm init 실행 시 정의하지 않음  

해결방법:  
```bash
kubeadm init --pod-network-cidr 10.244.0.0/16
```

### kubectl get services를 실행하였을때 external ip가 <pending>인 경우

원인:
```bash
kubectl get services
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP          12d
my-service   LoadBalancer   10.108.240.218   <pending>     8080:32071/TCP   4m8s
nginx        LoadBalancer   10.101.92.104    <pending>     80:32201/TCP     10d
```

해결방법:
```bash
#https://stackoverflow.com/questions/44110876/kubernetes-service-external-ip-pending
#kubectl patch svc <svc-name> -n <namespace> -p '{"spec": {"type": "LoadBalancer", "externalIPs":["172.31.71.218"]}}'
kubectl patch svc nginx -n default -p '{"spec": {"type": "LoadBalancer", "externalIPs":["192.168.0.4"]}}'
```

## 참고사이트:
- Kubernetes 설치:  
  + <https://futurecreator.github.io/2019/02/25/kubernetes-cluster-on-google-compute-engine-for-developers/>
- Kubernetes 이론:  
  + <https://subicura.com/2019/05/19/kubernetes-basic-1.html>
- kubernetes 명령어:  
  + <https://zzsza.github.io/development/2019/01/11/kubernetes-and-deployment/>
- kubenetes taints and tolerations:  
  + <https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/>  
  + <https://arisu1000.tistory.com/27846>
- kubernetes에서 사용하지 않는 이미지 삭제 방법:  
  + <https://www.ibm.com/support/knowledgecenter/en/SSBS6K_3.1.1/manage_images/remove_image.html>  
  + <https://stackoverflow.com/questions/51395040/manually-deleting-unused-images-on-kubernetes-gke>
- 외부 IP 주소로 서비스 노출하는 방법:
  + <https://kubernetes.io/ko/docs/tutorials/stateless-application/expose-external-ip-address/>