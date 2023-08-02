# 컨피그맵과 시크릿: 애플리케이션 설정
## 컨테이너화된 애플리케이션 설정
## 컨테이너에 명령줄 인자 전달
### 도커에서 명령어와 인자 정의
컨테이너에서 실행하는 전체 명령은 명령어와 인자 두 부분으로 구성되어 있다.

Dockerfile에서는 아래와 같이 두 부분을 정의한다.
- ENTRYPOINT: 컨테이너가 시작될 때 호출될 명령어를 정의
- CMD: ENTRYPOINT에 전달되는 인자를 정의

ENTRYPOINT 명령어로 실행하고 기본 인자를 정의하려는 경우에만 CMD를 지정하는 것이 권장되는 방법이다.

shell vs exec
- shell: ENTRYPOINT node app.js
- exec: ENTRYPOINT ["node", "app.js"]

### 쿠버네티스에서 명령과 인자 재정의
```
kind: Pod
spec:
  containers:
  - image: some/image
    command: ["/bin/command"]
    args: ["args1", "args2", "args3"]
```
위와 같이 쿠버네티스에서는 컨테이너를 정의할 때 command와 args를 이용하여 ENTRYPOINT, CMD를 각각 재정의한다.

## 컨테이너의 환경변수 설정
### 컨테이너 정의에 환경변수 지정
``` yaml
kind: Pod
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL
      value: "30"
    name: html-generator
...
```

### 변수 값에서 다른 환경변수 참조
``` yaml
env:
- name: FIRST_VAR
  value: "foo"
- name: SECOND_VAR
  value: "$(FIRST_VAR)bar"
```

## 컨피그맵
쿠버네티스에서는 설정 옵션을 컨피그맵이라 부르는 별도 오브젝트로 분리할 수 있다. 컨피그맵은 짧은 문자열에서 전체 설정 파일에 이르는 값을 가지는 키/값 쌍으로 구성된 맵이다.

### 컨피그맵 생성
$ kubectl create configmap fortune-config --from-literal=sleep-interval=25

컨피그맵 키는 유효한 DNS 서브도메인이어야 한다. (영문, 숫자, 대시, 밑줄, 점)

생성한 컨피그맵을 yaml로 출력
$ kubectl get configmap fortune-config -o yaml

yaml 파일을 쿠버네티스 API에 게시
$ kubectl create -f fortune-config.yaml

파일 내용으로 컨피그맵 생성
$ kubectl create configmap my-config --from-file=config-file.conf

### 컨피그맵 항목을 환경변수로 컨테이너에 전달
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortrune-env-from-configmap
spec:
  containers:
  - image: luksa/fortune:env
    env:
    - name: INTERVAL
      valueFrom:
        configMapKeyRef:
          name: fortune-config
          key: sleep-interval
```
- fortune-config의 sleep-interval 키에 설정된 값을 INTERVAL 환경변수로 선언

파드에 존재하지 않는 컨피그맵을 참조할 시 컨테이너는 시작하는데 실패한다. (configMapKeyRef.optional: true일 경우 제외)

### 컨피그맵의 모든 항목으 한 번에 환경변수로 전달
``` yaml
spec:
  containers:
  - image: some-image
    envFrom:
    - prefix: CONFIG_
    configMapRef:
      name: my-config-map
```
- env 대신 envFrom을 사용
- 모든 환경변수는 CONFIG_ 접두사를 가짐
- my-config-map 을 사용

### 컨피그맵 볼륨을 사용해 컨피그맵 항목을 파일로 노출

### 애플리케이션을 재시작하지 않고 애플리케이션 설정 업데이트
환경변수 또는 명령줄 인수를 설정 소스로 사용할 때의 단점은 프로세스가 실해오디고 있는 동안에 업데이트를 할 수 없다는 것이다.
컨피그맵을 사용해 볼륨으로 노출하면 파드를 다시 만들거나 컨테이너를 다시 시작할 필요 없이 설정을 업데이트할 수 있다.

컨피그맵 편집
$ kubectl edit configmap fortune-config
$ kubectl exec fortune-configmap-volume -c web-server cat /etc/nginx/conf.d/my-nginx-config.conf
nginx 자체가 파일 변경을 감시하지 않고 자동으로 다시 로드하지 않기 때문에 변화는 없다.

$ kubectl exec fortune-configmap-volume -c web-server -- nginx -s reload
위 명령어로 nginx 설정을 리로드하여 변경 확인

## 시크릿으로 민감한 데이터를 컨테이너에 전달
### 시크릿 소개
보안이 유지되어야 하는 자격 증명과 개인 암호화 키 같은 민감한 정보를 설정에 포함시킬 때 사용
시크릿 또한 컨피그맵과 같이 키-값의 맵으로 이루어져 있다.

### 기본 토큰 시크릿 소개
모든 파드에는 secret 볼륨이 자동으로 연결되어 있다.

$ kubectl describe secrets
``` bash
Name:        default-token-cfee9
Namespace:   default
Labels:      <none>
Annotations: kubernetes.io/service-account.name=default
             kubernetes.io/service-account.uid=cc04bb39-b53f-42010af00237
Type:        kubernetes.io/service-account-token

Data
====
ca.crt:      1139 bytes
namespace:   7 bytes
token:       ...
```

시크릿이 갖고 있는 세가지 항목(ca.crt, namespace, token)은 파드 안에서 쿠버네티스 API 서버와 통신할 때 필요한 모든 것을 나타낸다.

### 시크릿 생성
$ kubectl create secret generic fortune-https --from-file=https.key --from-file=https.cert --from-file=foo
- fortune-https 라는 이름의 generic 시크릿을 생성
- https.key, https.cert 파일을 포함하고 있으며, 각각 https 암복호화에 사용되는 개인 키, 인증서

### 컨피그맵과 시크릿 비교
``` bash
$ kubectl get secret fortune-https -o yaml
```
``` yaml
apiVersion: v1
data:
  foo: YmFyCg==
  https.cert: LS0tLSlCRUd:TiBDRV3USUZDQ0FURS0tLS0tCkl3SURCekNDQ...
  https.key: LS0tLSlCRUdJTiBSU0EgUF3DVkFURSBLRVktLS0tLQpNSUlFcE..
kind: Secret
...
```

``` bash
$ kubectl get configmap fortune-https -o yaml
```
``` yaml
apiVersion: v1
data:
  my-nginx-config.conf:
    server {
      ...
    }
  sleep-interval:
    25
kind: ConfigMap
...
```

- 시크릿 항목의 내용은 base64 인코딩 문자열로 표시
- 컨피그맵은 플레인 텍스트로 표시
- 시크릿의 최대 크기는 1MB
- 시크릿에서도 stringData 필드를 사용하여 플레인 텍스트로도 설정 가능하지만 쓰기 전용이므로 kubectl get 명령으로는 stringData가 아닌 base64 인코딩된 data 하위에 노출

### 파드에서 시크릿 사용
https를 활성화하도록 컨피그맵 수정
``` bash
$ kubectl edit configmap fortune-config
```
``` yaml
...
data:
  my-nginx-config.conf:
    server {
      listen 80;
      listen 443 ssl;
      server_name www.kubia-example.com;
      ssl_certificate certs/https.cert;
      ssl_certificate_key certs/https.key;
      ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
      ssl_ciphers HIGH:!aNULL:!MD5;
      location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
      }
    }
  sleep-interval:
...
```

fortune-https 시크릿을 파드에 마운트
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortune-https
spec:
  containers:
  - image: luksa/fortune:env
    name: html-generator
    env:
    - name: INTERVAL
      valueFrom:
        configMapKeyRef:
          name: fortune-config
          key: sleep-interval
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html     readonly: true
    - name: config
      mountPath: /etc/nginx/conf.d
      readonly: true
    - name: certs
      mountPath: /etc/nginx/certs/
      readonly: true
    ports:
    - containerPort: 80
    - containerPort: 443
  volumes:
  - name: html
    emptyDir: {}
  - name: config
    configMap:
    name: fortune-config
    items:
    - key: my-nginx-config.conf
      path: https.conf
  - name: certs
    secret:
      secretName: fortune-https
```

secret 볼륨은 시크릿 파일을 저장하는데 인메모리 파일시스템(tmpfs)을 사용한다.
tmpfs를 사용하는 이유는 민감한 데이터를 디스크에 저장하지 않기 위함이다.







