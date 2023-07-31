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
