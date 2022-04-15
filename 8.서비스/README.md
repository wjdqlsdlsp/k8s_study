# 서비스

동일한 서비스를 제공하는 Pod그룹의 단일 진입점을 제공

```yaml
apiVersion: v1
kind: Service
metadata:
	name: webui-svc
spec:
	clusterIP: 10.96.100.100
	selector:
		app: webui
	ports:
	-	protocol: TCP
		port: 80
		targetPort: 80
```



#### 서비스 타입

- ClusterIP : Pod 그룹의 단일 진입점 생성
- NodePort : ClusterIP 생성후, 모든 Worker Node에 외부에서 접속가능한 포트가 예약
- LoadBalancer : 클라우드나 오픈스택에 적용
- ExternalName : 클러스터 안에서 외부에 접속시 사용할 도메인 등록 및 사용 ( 클러스터 도메인이 실제 외부 도메인으로 치환되어 동작)



#### Kubernetes 헤드리스 서비스

ClusterIP 가 없는 서비스로, 단일 진입점이 필요 없을 때 사용

Pod의 endpoint로 DNS레코드가 생성됨



#### kube-proxy

쿠버네티스 서비스의 백엔드를 구현

endpoint 연결을 위한 iptables구성, nodePort로의 접근과 Pod연결을 구현
