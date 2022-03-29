# 쿠버네티스에서 컨테이너

> macOS 기준(m1 mac)으로 진행했습니다.



## 1. 기본 사용법

```shell
kubectl [command] [TYPE] [NAME] [flags]
```

기본 사용법은 위와 같습니다.

<br>

#### 파드 생성

```shell
kubectl run echoserver --image='k8s.gcr.io/echoserver:1.10' --port=8080
```

#### 파드 접근

```shell
kubectl expose po echoserver --type=NodePort
```

#### 파드 조회

```shell
kubectl get pods
```

#### 서비스 조회

```shell
kubectl get services
```

#### 포트 포워딩

```shell
kubectl port-forward svc/echoserver 8080:8080
```

#### (다른 터미널에서) 정상 작동확인 

```she
curl http://localhost:8080
```

#### 파드 삭제

```shell
kubectl delete pod echoserver
```

#### 서비스 삭제

```shell
kubectl delete service echoserver
```

<br>

#### 2. 디플로이먼트 이용하기

#### 디플로이먼트 생성

```shell
kubectl create deployment nginx-app --image nginx --port=80
```

#### 디플로이먼트 조회

```shell
kubectl get deployments
```

#### 파드 갯수 조정

```shell
kubectl scale deploy nginx-app --replicas=2
```

#### 디플로이먼트 삭제

```shell
kubectl delete deployment nginx-app
```

#### yaml 파일로 실행

```shell
kubectl apply -f nginx-app.yaml
```

yaml파일을 사전에 생성해야합니다.

```shell
kubectl expose deployment nginx-app --type NodePort
```

외부에서 접속하기 위해서는 위의 코드로 노출시켜야합니다.



