# 리소스 제한 & 환경변수 설정



#### 리소스 제한

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-resource
spec:
  containers:
  - name: nginx-container
    image: nginx:1.14
    ports:
    - containerPort: 80
      protocol: TCP
    resources:
      requests:
        cpu: 200m
        memory: 250Mi
      limits:
        cpu: 1
        memory: 500Mi
```

위의 yaml을 보면 resources 영역이 있는데 이를 통해서 메모리 제한을 줄 수있음.

Mi, Ki 등을 사용하는데 이는 Mib, Kib를 의미

cpu 단위에서는 m이 나오는데 이는 1코어 == 1000m 를 의미하여 200m 는 20프로 사용을 의미 단위가 없으면 core 

만약 requests를 만족하는 노드가 없을 경우(설정한 자원을 할당 할 수없는 경우), 계속해서 pending상태에 있을 수 있습니다.

<br>

#### 환경 변수설정

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-env
spec:
  containers:
  - name: nginx-container
    image: nginx:1.14
    ports:
    - containerPort: 80
      protocol: TCP
    env:
    - name: MYVAR
      value: "testvalue"
```

위의 코드를 보면 env를 설정했는데, 이는 각각 환경변수, value를 의미합니다.

```shell
kubectl exec [pod이름] -it -- /bin/bash

env
```

위의 명령어를 통해서 해당 pod로 접속하고  env명령어를 통해 env가 잘 등록되어 있음을 확인할 수 있습니다.

<br>

#### pod 실행패턴

- Sidecar : 하나의 파드안에, 단독으로 동작을 구현하지 못하고, 컨테이너 2개이상이 함께 동작해야하는 패턴
- Adapter : 중간에 어댑터 역할을 하는 패턴 ( Sidecar는 안에서 자기들 끼리 통신을한다면, Adapter는 어탭터를 통해 밖과 소통하는 방식 )
- Ambassador : 데이터를 캐시로 로드밸런싱하는 패턴
