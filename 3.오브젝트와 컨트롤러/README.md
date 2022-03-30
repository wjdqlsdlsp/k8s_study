# 오브젝트와 컨트롤러

> macOS 기준(m1 mac)으로 진행했습니다.

## 1. 네임스페이스

네임스페이스는, 클러스터 하나를 여러 개 논리적인 단위로 나눠서 사용하는 것 입니다.



#### 네임스페이스 조회

```shell
☁  ~  kubectl get namespaces
NAME              STATUS   AGE
default           Active   23h
kube-node-lease   Active   23h
kube-public       Active   23h
kube-system       Active   23h
```

- default - 기본 네임스페이스
- kube-system - 시스템 관리 네임스페이스
- kube-public - 클러스터 안 모든 사용자가 읽을 수 있는 네임스페이스
- kube-node-lease - 각 노드의 임대 오브젝트를 관리하는 네임스페이스



#### config current-context 조회

```shell
☁  ~  kubectl config current-context
docker-desktop
```



#### config get-contents

```shell
☁  ~  kubectl config get-contexts docker-desktop
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop   docker-desktop   docker-desktop
```

namespace 를 지정하지 않았을 경우 위와 같이 공백으로 남아 있음.



#### set-context

```shell
☁  ~  kubectl config set-context docker-desktop --namespace=kube-system
Context "docker-desktop" modified.
```

위와 같이 set-context를 통해 namespace를 지정할 수 있음

```shell
☁  ~  kubectl config get-contexts $(kubectl config current-context) --namespace=kube-system
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop   docker-desktop   docker-desktop   kube-system
```

위와 같이 환경변수를 통한 지정을 할 수 있습니다.

```shell
☁  ~  kubectl config view | grep namespace
    namespace: kube-system
```

위의 코드를 이용하면 namespace를 확인 할 수 있음.

#### 전체 namespace 확인

```shell
☁  ~  kubectl get pods --all-namespaces
NAMESPACE     NAME                                     READY   STATUS    RESTARTS       AGE
kube-system   coredns-78fcd69978-gv4g9                 1/1     Running   3 (40m ago)    23h
kube-system   coredns-78fcd69978-rbr4j                 1/1     Running   3 (40m ago)    23h
kube-system   etcd-docker-desktop                      1/1     Running   3 (40m ago)    23h
kube-system   kube-apiserver-docker-desktop            1/1     Running   3 (40m ago)    23h
kube-system   kube-controller-manager-docker-desktop   1/1     Running   3 (40m ago)    23h
kube-system   kube-proxy-jxmx2                         1/1     Running   3 (40m ago)    23h
kube-system   kube-scheduler-docker-desktop            1/1     Running   3 (40m ago)    23h
kube-system   storage-provisioner                      1/1     Running   4 (39m ago)    23h
kube-system   vpnkit-controller                        1/1     Running   10 (14m ago)   23h
```

```shell
☁  ~  kubectl config set-context $(kubectl config current-context) --namespace=default
Context "docker-desktop" modified.
```

이후 실습을 따라 다시 default값으로 변경 하였습니다.

<br>

## 2. 템플릿



#### yaml 파일 생성하기

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-simple-pod
  labels:
    app: kubernetes-simple-pod
spec:
  containers:
  - name: kubernetes-simple-pod
    image: arisu1000/simple-container-app:latest
    ports:
    - containerPort: 8080
```

위와 같이 템플릿 코드를 작성하고 이를 통해 파드 구축



#### 템플릿 코드로 파드 구축

```shell
kubectl apply -f pod-sample.yaml
```

위에서 작성한 템플릿 파일을 apply -f 를 통해서 실행



```shell
⚡  kubectl get pod
NAME                    READY   STATUS    RESTARTS   AGE
kubernetes-simple-pod   1/1     Running   0          119s
```

정상적으로 실행됨을 알 수 있음.
