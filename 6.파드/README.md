# 리소스 제한 & 환경변수 설정



#### 컨트롤러

- 파드의 개수를 보장



#### Replication Controller

- 요구하는 파드의 개수를 보장하는 것을 목표로 하는 가장 기본적인 컨트롤러

기본구성

- selector
- replicas
- template

```yaml
apiVersion: v1
kind: RepicationController
metadata:
	name: <RC_이름>
spec:
	replicas: <갯수>
	selector:
		key: value
	template:
		<컨테이너 템플릿>
```

selector 에 있는  key - value를 가지는 파드를 설정한 replicas 갯수만큼 갯수를 유지하는 것을 목표로 작동하며, 갯수가 부족할 경우 template으로 설정한 컨테이너 템플릿을 실행하여 갯수를 보장



```shell
kubectl create rc -exam --image=nginx --replicas=3 --selector=app=webui
```

위와 같이, shell에서 옵션을 통해 설정가능



#### label 설정

```yaml
apiVersion: v1
kind: RepicationController
metadata:
	name: rc-nginx
spec:
	replicas: 3
	selector:
		app: webui
	template:
		metadata:
			name: nginx-pod
			labels:
				app:webui
		specL
			containers:
			-	name: nginx-container
				image: nginx:1.14
```

위와 같이, selector를 보고 replicas갯수만큼 유지 ( template에 labels에 selector에 해당하는 내용이 없으면 에러 발생)

