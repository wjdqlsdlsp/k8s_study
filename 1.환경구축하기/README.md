# 환경 구축하기

> GCP 기준으로 진행했습니다.



## 1. VM 인스턴스 생성 (5개 생성)

총 5개의 인스턴스를 생성하며, 1,2,3 인스턴스는 마스터 인스턴스 4,5 인스턴스는 워크 노드로 사용할 예정

<br>

## 2. SSH 접속 허용

instance-1 에 SSH접속 이후, 다른 instance에 접속하도록 허용하기 위해 ssh키를 다운 받아야함.

```shell
ssh-keygen -t rsa
```

위의 커맨드를 통해 ssh 키 생성

```shell
ls -al .ssh/
```

ssh의 생성 확인

```shell
cat .ssh/id_rsa.pub
```

위의 코드를 통해 ssh코드를 출력하며 위의 키를 복사

이후 GCP 메타데이터 SSH를 이용하여 간편하게 다른 인스턴스에 접속할 수 있도록 하용

```shell
ssh instance-2 hostname
```

위의 코드를 통해 연결이 가능한 상태인지 확인 ( yes입력 )

## 3. Kubespray 설치

```shell
sudo apt update
sudo apt-get install python-pip
sudo apt-get install python3-pip
```

Instance-1 을 접속하여 위의 코드로 update 및 pip install

```shell
git clone https://github.com/kubernetes-sigs/kubespray
```

kubespary git clone 진행

```shell
cd kubespray
ls -al
cat requirements.txt
```

파일이 잘 다운로드 되어있는지 확인

```shell
sudo pip3 install -r requirements.txt
cp -rfp inventory/sample inventory/mycluster

ls inventory/mycluster
```

https://github.com/kubernetes-sigs/kubespray 해당 깃헙을 통해 Ansible 다운로드 과정 진행

(위의 과정만 진행하였습니다.)

```shell
vi inventory/mycluster/inventory.ini
```

vim 을 이용하여 ini 파일 수정

```shell
[all]
instance-1 ansible_host=10.182.0.2 ip=10.182.0.2 etcd_member_name=etcd1
instance-2 ansible_host=10.182.0.3 ip=10.182.0.3 etcd_member_name=etcd2
instance-3 ansible_host=10.182.0.4 ip=10.182.0.4 etcd_member_name=etcd3
instance-4 ansible_host=10.182.0.5 ip=10.182.0.5
instance-5 ansible_host=10.182.0.6 ip=10.182.0.6

[kube_control_plane]
instance-1
instance-2
instance-3

[etcd]
instance-1
instance-2
instance-3

[kube_node]
instance-4
instance-5

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

ansible_host 및 ip를 체크하세요

```shell
ansible-playbook -i inventory/mycluster/inventory.ini -v --become --become-user=root cluster.yml
```

약 20분정도 시간 소요

[output]

```shell
PLAY RECAP ****************************************************************************************************
instance-1                 : ok=733  changed=146  unreachable=0    failed=0    skipped=1241 rescued=0    ignored=5   
instance-2                 : ok=662  changed=132  unreachable=0    failed=0    skipped=1080 rescued=0    ignored=3   
instance-3                 : ok=664  changed=133  unreachable=0    failed=0    skipped=1078 rescued=0    ignored=3   
instance-4                 : ok=514  changed=96   unreachable=0    failed=0    skipped=728  rescued=0    ignored=1   
instance-5                 : ok=514  changed=96   unreachable=0    failed=0    skipped=727  rescued=0    ignored=1   
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Sunday 27 March 2022  09:10:14 +0000 (0:00:00.487)       0:31:55.888 ********** 
=============================================================================== 
kubernetes/preinstall : Install packages requirements ------------------------------------------------- 33.40s
kubernetes/control-plane : kubeadm | Initialize first master ------------------------------------------ 31.48s
kubernetes/control-plane : Joining control plane node to the cluster. --------------------------------- 25.95s
kubernetes-apps/ansible : Kubernetes Apps | Start Resources ------------------------------------------- 23.24s
download : download | Download files / images --------------------------------------------------------- 22.02s
container-engine/containerd : containerd | Unpack containerd archive ---------------------------------- 20.55s
container-engine/crictl : extract_file | Unpacking archive -------------------------------------------- 19.60s
container-engine/nerdctl : extract_file | Unpacking archive ------------------------------------------- 19.56s
container-engine/nerdctl : extract_file | Unpacking archive ------------------------------------------- 18.99s
container-engine/nerdctl : download_file | Download item ---------------------------------------------- 18.14s
container-engine/containerd : download_file | Download item ------------------------------------------- 18.12s
container-engine/nerdctl : download_file | Download item ---------------------------------------------- 17.61s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates -------------------------------- 16.66s
download : download | Download files / images --------------------------------------------------------- 16.65s
kubernetes/preinstall : Preinstall | wait for the apiserver to be running ----------------------------- 16.49s
network_plugin/calico : Start Calico resources -------------------------------------------------------- 16.02s
container-engine/containerd : download_file | Validate mirrors ---------------------------------------- 15.90s
container-engine/nerdctl : download_file | Validate mirrors ------------------------------------------- 15.85s
container-engine/nerdctl : download_file | Validate mirrors ------------------------------------------- 15.57s
download : download | Download files / images --------------------------------------------------------- 15.28s
```

위와 같이 failed=0가 표시되면 에러없이 환경이 잘 구축됨

```shell
sudo -i
```

클러스터 구성완료 이후, root계정으로 kubectl 명령어를 사용가능

```shell
kubectl get node
```

위의 코드로 현재 노드 상태를 확인 할 수 있음.