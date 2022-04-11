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
kind: ReplicationController
metadata:
  name: rc-nginx
spec:
  replicas: 3
  selector:
    app: web
  template:
    metadata:
      name: nginx-pod
      labels:
        app: web
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14
```

위와 같이, selector를 보고 replicas갯수만큼 유지 ( template에 labels에 selector에 해당하는 내용이 없으면 에러 발생)



```shell
kubectl get replicationcontrollers

# kubectl get rc
```

위의 코드로 현재 요구되는 replicas 정보를 확인할 수 있습니다.



```shell
kubectl describe rc rc-nginx
```

조금더 자세히 보고싶으면 위와같은 코드를 이용하면 됩니다.



```shell
kubectl scale rc rc-nginx --replicas=2
```

위의 코드를 통해 replicas를 조정할 수 있습니다.



#### ReplicaSet

ReplicationController 와 같은 컨트롤 역할을 하지만, 보다 풍부한 selector를 지원

```yaml
selector:
	matchLabels:
		component: redis
	matchExpressions:
	-	{key: tier, operator: In, values: [cache]}
	- {key: environment, operatorL NotIn, values: [dev]}
```

위와같은 좀더 다양한 seletor를 지원합니다.



#### matchExpressions 연산자

In: key와 values를 지정하여 key, value가 일치하는 Pod만 연결

NotIn: 일치하지 않는 파드에 연결

Exists: key에 맞는 label의 파드를 연결

DoesNotExist: key와 다른 label의 파드를 연결



```yaml
ReplicaSet
spec:
	replicas: 3
	selector:
		matchLabels:
			app: webui
		matchExpressions:
		-	{key: version, operator: In, value: ["2.1", "2.2"]}
```

위와 같이, 조건을 줄 수 있습니다.



```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web 
  template:
    metadata:
      name: nginx-pod
      labels:
        app: web
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.14 
```



```shell
☁  ~  kubectl get pod --show-labels
NAME             READY   STATUS    RESTARTS   AGE   LABELS
rs-nginx-7fvws   1/1     Running   0          30s   app=web
rs-nginx-jrjhf   1/1     Running   0          30s   app=web
rs-nginx-njsvl   1/1     Running   0          30s   app=web

☁  ~  kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
rs-nginx   3         3         3       65s

☁  ~  kubectl scale rs rs-nginx --replicas=2
replicaset.apps/rs-nginx scaled
☁  ~  kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
rs-nginx   2         2         2       2m16s
```

위의 yaml파일을 통해서  생성하고 확인하면 정상적으로 작동함을 알 수 있습니다.

```shell
☁  ~  kubectl delete rs rs-nginx --cascade=false
warning: --cascade=false is deprecated (boolean value) and can be replaced with --cascade=orphan.
replicaset.apps "rs-nginx" deleted
```

--cascade 옵션을 False로 설정함으로서, 연쇄삭제를 막을 수 있습니다. ( 파드는 삭제 x )

