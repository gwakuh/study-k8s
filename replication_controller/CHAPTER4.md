# 레플리케이션과 그 밖의 컨트롤러: 관리되는 파드 배포

쿠버네티스에서 파드는 배포 가능한 기본 단위이다.
레플리케이션컨트롤러 또는 디플로이먼트와 같은 유형의 리소스를 생성해 파드를 관리한다.

## 파드를 안정적으로 유지하기

### 파드의 컨테이너 중 하나가 죽으면?
파드가 노드에 스케줄링되면, 해당 노드의 Kubelet은 파드의 컨테이너를 실행하고, 파드가 존재하면 컨테이너가 계속 실행되게 한다.

## Liveness probe
컨테이너가 살아있는지 확인하는 health-check의 일환
Pod specification에 각 컨테이너의 Liveness probe를 설정할 수 있음
쿠버네티스는 주기적으로 프로브를 실행하고 실패할 경우 컨테이너를 재시작한다.
Readiness probe와는 쓰임새가 다름

### Mechanism
- HTTP GET Probe
  - IP, Port, Path를 지정하여 HTTP GET Request
  - HTTP Status 2xx or 3xx인 경우 성공으로 간주
  - 이 외의 응답 코드 혹은 Timeout 시 실패한 것으로 간주하고 컨테이너를 재시작
- TCP Socket Probe
  - Port를 지정하여 TCP Connect
  - 연결에 성공하면 성공 아니면 실패 후 컨테이너 재시작
- Exec Probe
  - 컨테이너 내 명령을 실행하고 Return 0 이면 성공 아니면 실패 후 컨테이너 재시작

### HTTP based Liveness Probe
``` kubia-liveness-probe.yaml
apiVersion: v1
kind: pod
metadata:
  name: kubia-liveness
spec:
  containers:
  - image: gwakuh/kubia-unhealthy
    name: kubia
	livenessProbe:
    httpGet:
	  path: /
      port: 8080
```
주기적으로 "http://localhost:8080/" 주소로 GET 요청을 보내 정상 동작하는지 확인

$ kubectl describe po kubia-liveness
Name: kubia-liveness
Containers:
  kubia:
    ...
    State: running
      Started: Sun, 14 May 2017 11:41:40 +0200 
    Last State: Terminated
      Reason: Error
      Exit Code: 137
    Ready: True
	Restart Count: 1
    Liveness: http-get http://:8080/ delay=0s timeout=1s
	          period=10s #success=1 #failure=3
    ...

컨테이너가 현재 실행 중이지만 이전에 에러가 발생해 종료되었음을 나타냄
Exit Code 137은 128 + x를 뜻하며, 이는 프로세스가 외부 신호에 의해 종료되었음을 나타냄
128 +9 즉 9는 시그널 넘버 SIGKILL(9)를 뜻하며, 프로세스가 강제로 종료되었음을 의미

### Liveness Probe 추가 속성
- delay
- timeout
- period
- success
- failure
- initialDelaySeconds (첫 번째 프로브 실행 전 대기시간)

- 위 예제로 설명하면 컨테이너가 시작한 후 즉시 프로브가 시작 (delay=0s)
- 프로브는 10초 마다 실행 (period=10s)
- 1초 안에 응답해야 성공 (timeout=1s)
- 1번 성공하면 성공 (#success=1)
- 3번 연속으로 실패할 시 컨테이너 재시작 (#failure=3)

## 레플리케이션컨트롤러(ReplicationController; RC)
RC에 의해 실행된 파드는 항상 실행되도록 보장한다.

### 특징
- 실행 중인 파드 목록을 지속적으로 모니터링
- 파드의 수가 설정된 수와 일치하는지 항상 확인
- 기존 파드가 사라지면 새 파드를 실행해 파드가 항상 실행되도록 유지
- 클러스터 노드에 장애가 발생하면 해당 노드에서 실행 중인 모든 파드를 교체
- 수동 또는 자동으로 파드를 수평적 확장

### 필수 요소
- Label Selector: RC의 범위에 있는 파드를 결정
- Replica Count: 실행할 파드의 의도하는 수를 지정
- Pod Template: 새로운 파드 레플리카를 만들 때 사용

### 수평 파드 스케일링
1. 명령어로 직접 조정
$ kubectl scale rc kubia --replicas=10

2. 텍스트 편집기로 수정
환경변수 KUBE_EDITOR에 설정된 편집기로 열림
$ kubectl edit rc kubia
```
apiVersion: v1
kind: ReplicationController 
metadata:
...
spec:
replicas: 3 // 3 -> 10
selector:
app: kubia
...
```

$ kubectl get rc

### ReplicationController 삭제
kubectl delete를 통해 삭제 시 파드도 함께 삭제된다.
하지만, RC를 통해 생성한 파드는 RC의 관리만 받기 때문에 RC만 삭제하고 파드는 실행 상태로 둘 수 있다. (--cascade=false 옵션을 사용하여 관리되지 않는 파드로 만듦)
$ kubectl delete rc kubia --cascade=false

## ReplicaSet
RC 이후에 도입된 RC와 유사한 리소스, 차세대 RC

### RS vs RC
- 좀 더 풍부한 표현식을 사용하는 파드 셀렉터
  - RC는 특정 레이블의 파드만 매칭
  - RS는 서로 다른 레이블의 파드도 매칭시켜 하나의 그룹으로 취급할 수 있음
  - RS는 env=\*도 가능

### RS Example
``` kubia-replicaset.yaml
apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    matchLabels:
	  app: kubia
  template:
    metadata:
	  labels:
	    app: kubia
    spec:
	  containers:
	  - name: kubia
	    image: luksa/kubia
```

- RS는 apps 그룹의 v1beta2 버전에 속함
- RC와 유사한 matchLabels 셀렉터를 사용
- 템플릿은 RC와 동일

### RS 생성 및 검사
$ kubectl create -f kubia-replicaset.yaml

$ kubectl get rs

$ kubectl describe rs

### RS의 Label Selector
``` kubia-replicaset-matchexpressions.yaml
selector:
  matchExpressions:
    - key: app
	  operator: In
	  values:
	    - kubia
```

- 파드의 키가 app인 레이블을 포함해야 한다.
- 레이블의 값은 kubia 여야 한다.

4가지 Operator
- In: 레이블의 값이 지정된 값 중 하나와 일치해야 한다.
- NotIn: 레이블의 값이 지정된 값과 일치하지 않아야 한다.
- Exists: 지정된 키를 가진 레이블이 포함되어야 한다. value 필드를 지정하지 않아야 한다.
- DoesNotExist: 지정된 키를 가진 레이블이 포함되어 있지 않아야 한다. value 필드를 지정하지 않아야 한다.

### RS 삭제
$ kubectl delete rs kubia

## DaemonSet
모든 노드에서 정확히 하나의 파드를 실행할 때 사용
RS 혹은 RC가 클러스터에 원하는 수의 파드 복제본이 존재하는지 확인하는 반면, 데몬셋은 원하는 복제본의 수라는 개념이 없다. 파드 셀렉터와 일치하는 파드 하나가 각 노드에서 실행 중인지 확인하는게 데몬셋의 역할이다.

### DaemonSet으로 특정 노드에서만 파드 실행하기
파드 템플릿에서 node-Selector 속성을 지정하여 실행 시 특정 노드에서만 실행되며, 특별히 지정하지 않을 경우 모든 노드에서 실행한다.

## Completable Task
계속 실행되어야 하는 파드와 다르게 작업을 완료 후 종료되며, 재시작하지 않는다.

### Job resource

## Summary
- 컨테이너가 정상적이지 않으면 재시작하도록 하는 LivenessProbe를 지정할 수 있다.
- 파드는 실수로 삭제되거나 실행 중인 노드에 장애가 발생하거나 노드에서 퇴출되면 다시 생성되지 않기 때문에 파드를 직접 생성하면 안된다.
- RC는 의도하는 수의 파드 replica를 항상 실행 상태로 유지한다.
- 파드를 수평적 스케일링하려면 RC의 Replica Count를 변경하면 된다.
- 파드는 RC에 종속적이지 않고, RC간 이동이 가능하다.
- RC는 파드 템플릿에서 새로운 파드를 생성한다. 템플릿을 변경해도 기존 파드에 영향을 주지 않는다.
- RC는 RS 혹은 Deployment로 교체해야 하며, RS, Deployment는 동일한 기능을 제공하면서 RC보다 강력한 기능을 제공한다.
- RC와 RS는 클러스터 노드에 파드를 스케줄링하는 반면, DaemonSet은 모든 노드에 각각 하나의 파드를 실행하도록 한다.
- 배치 작업을 수행하는 파드는 k8s의 Job 리소스로 생성해야 한다. 직접 생성하거나 RC와 유사한 오브젝트로 생성하면 안된다.
- 언젠가 실행해야 하는 잡은 크론잡 리소스를 통해 생성할 수 있다.

 



