# Deployment



- ReplicaSet을 컨트롤해서 Pod수를 조절

- Rolling Update & Rolling Back 

Deployment 가 ReplicaSet를 컨트롤하고, ReplicaSet이 pod를 컨트롤하는 구조



#### Rolling Update & Rolling Back

- 파드 인스턴스를 점진적적으로 새로운 버전으로 업데이트하여, 디플로이먼트 업데이트가 서비스 중단없이 진행

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx
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
NAME                               READY   STATUS    RESTARTS   AGE
pod/deploy-nginx-74dbbbc58-8scxh   1/1     Running   0          40s
pod/deploy-nginx-74dbbbc58-f2gmt   1/1     Running   0          40s
pod/deploy-nginx-74dbbbc58-wbnw5   1/1     Running   0          40s
# <pod>/<deployment이름>-<ReplicaSet>-<pod>의 이름 형식을 가짐
```

디플로이먼트가 레플리카셋을 컨트롤하고 있기 때문에, 레플리카셋을 삭제하더라도 다시 생성



#### Rolling update

```shell
kubectl set image deployment <deploy_name> <container_name>=<new_version_image>
# kubectl set image deployment app-deploy app=nginx:1.15 --record 
```

위의 롤링업데이트 과정을 통해 컨테이너를 교체할 수 있습니다.



```shell
kubectl rollout status deployment app-deploy
```

위의 rollout status를 통해서 현재 상태 정보를 파악할 수 있습니다.

```shell
kubectl rollout pause deployment app-deploy
```

업데이트 중지합니다.

```shell
kubectl rollout resume deployment app-deploy
```

중지된 업데이트 다시 진행합니다.

```shell
kubectl rollout history deployment app-deploy
```

롤아웃 히스토리를 조회할 수 있습니다.



```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-nginx
  annotations:
  	kubernetes.io/change-cause: version 1.14
spec:
	progressDeadlineSeconds: 600
	revisionHistoryLimit: 10
	strategy:
		rollingUpdate:
			maxSurge: 25%
			maxUnavailable: 25%
		type: RollingUpdate
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

progressDealineSeconds를 통해서 설정한 초안에 업데이트를 진행하지 못하면, 업데이트를 취소 시키게 설정할 수 있습니다.

revisionHistoryLimit 값을 설정할 수 있습니다.

maxSurge 설정을 통해서 업데이트를 진행할 때, 추가 레플리카를 몇개까지 유지하는지 설정

maxUnavailable 세팅을 통해 중단되는 갯수를 조절

RollingUpdate를 통해 바로 업데이트를 설정



```shell
kubectl rollout undo deployment app-deploy

# kubectl rollout undo deployment app-deploy --to-revision=3 
```

다음 코드를 통해 롤백이 가능합니다. ( revision 옵션을통해 원하는 버전으로 이동 가능 )



#### DaemonSet

전체 노드에서 pod가 한 개씩 실행되도록 보장

로그 수집기 및 모니터링 에이전트와 같은 프로그램 실행시 적용.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-nginx
spec:

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

RelicaSet과 비교했을 때, replica 옵션만 없음.



#### Statefulset

파드의 상태를 유지해주는 컨트롤러

- 파드 이름
- 파드 볼륨

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sf-nginx
spec:
	replica: 3
	serviceName: sf-nginx-service
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

ReplicaSet과 구성이 비슷하지만 특별하게 serviceName이 포함되어 있습니다.



#### Job

```shell
☁  ~  kubectl run testpod --image=centos:7 --command sleep 5
pod/testpod created
```

다음과 같이 sleep 5를 커맨드로 주면, 5초뒤 삭제 하고 run시키고를 반복합니다.

쿠버네티스는 기본적으로 running 중인 파드를 보장합니다. ( 쿠버네티스의 동작 방식 )

- 쿠버네티스는 Pod를 running 중인 상태로 유지
- Batch 처리하는 Pod는 작업이 완료되면 종료됨
- Batch 처리에 적합한 컨트롤러로 Pod의 성공적인 완료를 보장

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-example
spec:
  template:
    spec:
      containers:
      - name: centos-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "echo 'Hello World'; sleep 5; echo 'Bye'"
      restartPolicy: Never
```



#### CronJob

사용자가 원하는 시간에 Job 실행 예약 지원

CronJob Shedule의 경우 Linux에 있는 Crontab과 동일 ("0 3 1 * * ")

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```



```shell
☁  ~  kubectl get cronjob
NAME    SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   * * * * *   False     0        21s             3m29s

☁  ~  kubectl delete cronjob hello
cronjob.batch "hello" deleted
```

cronjob을 삭제하고 싶으면, 위와같이 delete를 통해 삭제



