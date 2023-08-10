# Exam1
```sh
# kubectl get nodes -o json > exam/json-sean.json
```

# Exam2
```sh
# kubectl create namespace exam-sean
```

# Exam3
```sh
# kubectl create deployment sean-deployment --image=nginx --replicas=2 --namespace=exam-sean
```

# Exam4
```sh
# kubectl get deployments --namespace=exam-sean -o custom-columns='DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[0].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace'

DEPLOYMENT        CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE
nginx-deploy      nginx:1.17        1                exam-sean
sean-deployment   nginx             2                exam-sean
```

# Exam5
```sh
#kubectl create deployment nginx-deploy --namespace=exam-sean --replicas=1 --image=nginx:1.16
#kubectl set image deployments/nginx-deploy nginx=nginx:1.17
```

# Exam6
```sh
# cat web-application.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-application
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: simple-webapp
  ports:
    - port: 8088
      nodePort: 31112
```
```sh
# kubectl create -f web-application.yaml
```

# Exam7
```sh
# kubectl expose deployment sean-deployment --name=sean-service --port=9090 --namespace=exam-sean
```

# Exam8
```sh
# cat sean-deployment.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sean-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: sean-pod
  template:
    metadata:
      labels:
        app: sean-pod
    spec:
      containers:
      - name: sean-pod
        image: nginx
```
```sh
# kubectl apply -f sean-deployment.yaml
```

위 sean-deployment.yaml 파일에서 replicas를 1로 변경 후 위 명령어 다시 실행해서 replica를 1로 반영

```sh
# kubectl create configmap sean-config --from-literal=hello=world --from-literal=drink=good --from-literal=happy=work
```
```sh
# vi sean-deployment.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sean-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sean-pod
  template:
    metadata:
      labels:
        app: sean-pod
    spec:
      containers:
      - name: sean-container
        image: nginx
        volumeMounts:
        - name: config-volume
          mountPath: /config
      volumes:
      - name: config-volume
        configMap:
          name: sean-config
```
```sh
# kubectl apply -f sean-deployment.yaml
```

# Exam9
```sh
# kubectl get pods

NAME                               READY   STATUS    RESTARTS   AGE
nginx-deploy-c848b6868-ttknz       1/1     Running   0          6d23h
sean-deployment-5b94cb584d-s64l8   1/1     Running   0          4m49s

# kubectl exec -it sean-deployment-5b94cb584d-s64l8 -- /bin/sh

# echo "Hello World!" > hello.txt

# cat hello.txt

Hello World!
```

# Exam10
```sh
# vi myapp-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'sleep 20']
  containers:
  - name: myapp-container
    image: nginx
```

```sh
# kubectl apply -f myapp-pod.yaml

# kubectl get pods myapp-pod
NAME        READY   STATUS     RESTARTS   AGE
myapp-pod   0/1     Init:0/1   0          19s

# kubectl get pods myapp-pod
NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   1/1     Running   0          23s
```