# 기본적인 명령어 익히기



```shell
kubectl --help
kubectl command -help

kubectl run <자원이름> <옵션>
kubectl create -f obj.yaml
kubectl apply -f obj.yaml

kubectl get <자원이름> <객체이름>
kubectl edit <자원이름> <객체이름>
kubectl describe <자원이름> <객체이름>

kubectl delete pod main
```



#### run vs create

둘 다 비슷한 동작을 수행하지만, create 의 경우, --replicas 옵션을 통해 1개 이상의 이미지를 실행 시킬 수 있음.



#### exec

```shell
kubectl exec [자원이름] -it -- /bin/bash
```

위의 코드를 통해 컨테이너 내부로 접속 가능



#### logs

```shell
kubectl logs [자원이름]
```



#### edit

```shell
kubectl edit deployments.apps [자원이름]
```

Yaml 파일 수정 가능 



```shell
kubectl run webserver --image=nginx:1.14 --port 80 --dry-run -o yaml > webserver-pod.yaml
```

k8s가 사용하는 yaml 다운로드하는 방법

```shell
kubectl create -f webserver-pod.yaml
```

이후 yaml로 쿠버네티스 실행하고 싶으면 위의 코드로 실행가능

```shell
kubectl get namespaces
```

현재 시스템에 namespaces 확인

이전까지 사용했던 코드들은 namespace를 지정하지 않았기 때문에, base(초기값 default)로 실행 

```shell
kubectl get pods -n [namespace]
```

해당 namespace의 pods만 조회

```shell
kubectl edit pod [파드이름]
```

파드의 yaml파일을 접근할 수 있게 vi 로 접근



#### livenessProbe

일종의 health check방식으로, 쿠버네티스가 상태를 확인하는 방법

```yaml
spec:
	containers:
	-name: nginx-container
	image: nginx:1.14
	livenessProbe:
		httpGet:
			path: /
			port: 80
		periodSeconds: 10
    successThreshold: 1
    timeoutSeconds: 1
```

위와같이, yaml 파일에서 livenessProbe를 지정해줌

#### httpGet

```yaml
livenessProbe:
	httpGet:
		path: /
		port: 80
```

해당 주소와 포트에 get 요청을 보내고 정상 응답이 오지않을 시, 컨테이너 재시작

#### tcpSocket

```yaml
livenessProbe:
	tcpSocket:
		port: 22
```

#### exec

```yaml
livenessProbe:
	exec:
		command:
		- ls
		- /data/file
```

