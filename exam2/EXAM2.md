# Example 

1. Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod.

```sh
vi nginx.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sean-nginx
spec:
  containers:
  - name: sean-nginx-container
    image: nginx
    env:
    - name: var1
      value: "var1"
```
```sh
$ kubectl apply -f nginx.yaml
$ kubectl exec sean-nginx -- env | grep var1
```

2. Create an nginx pod and exec into containers and verify that main.txt exist.
```sh
vi nginx.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sean-nginx
spec:
  containers:
  - name: sean-nginx-container
    image: nginx
    env:
    - name: var1
      value: "var1"
```
```sh
$ kubectl apply -f nginx.yaml
$ kubectl exec -it sean-nginx -- ls main.txt
ls: cannot access 'main.txt': No such file or directory
command terminated with exit code 2
```

3. Create a Pod with main container busybox and which executes this "while true; do echo `Hi I am from Main container' >> /var/log/index.html; sleep 5; done" and with sidecar container with nginx image which exposes on port 80. Use emptyDir Volume and mount this volume on path /var/log for busybox and on path /usr/share/nginx/html for nginx container. Verify both containers are running. 

```sh
$ vi busybox.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sean-busybox
spec:
  containers:
  - name: sean-busybox-container
    image: busybox
    command: ['sh', '-c', 'while true; do echo "Hi I am from Main container" >> /var/log/index.html; sleep 5; done']
    volumeMounts:
    - mountPath: /var/log
      name: log-volume
  - name: nginx-container
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: log-volume
  volumes:
  - name: log-volume
    emptyDir:
      sizeLimit: 500Mi
```
```sh
$ kubectl apply -f busybox.yaml
```

4. Check logs of each container that "busyboxpod-{1,2,3}"
```sh
$ kubectl logs sean-busybox3 -c sean-busybox1
bin
dev
etc
home
lib
lib64
proc
root
sys
tmp
usr
var

$ kubectl logs sean-busybox3 -c sean-busybox2
Hello World

$ kubectl logs sean-busybox3 -c sean-busybox3
this is the third container
```

5. Create a Pod with three busy box containers with commands "ls; sleep 3600;", "echo Hello World; sleep 3600;" and "echo this is the third container; sleep 3600" respectively and check the status
```sh
$ vi 5.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sean-busybox3
spec:
  containers:
  - name: sean-busybox1
    image: busybox
    command: ['sh', '-c', 'ls; sleep 3600;']
  - name: sean-busybox2
    image: busybox
    command: ['sh', '-c', 'echo Hello World; sleep 3600;']
  - name: sean-busybox3
    image: busybox
    command: ['sh', '-c', 'echo this is the third container; sleep 3600']
```
```sh
$ kubectl apply -f 5.yaml
```

6. Create a redis pod, and have it use a non-persistent storage Note: In exam, you will have access to kubernetes.io site, Refer : https://kubernetes.io/docs/tasks/configure-pod-container/configurevolume-storage/
```sh
$ vi 6.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```

7. Change the Image version back to 1.17.1 for the pod you just updated and observe the changes

8. Change the Image version to 1.15-alpine for the pod you just created and verify the image version is updated.

9. Create the nginx pod with version 1.17.4 and expose it on port 80

10. Create a redis pod and expose it on port 6379

11. Delete the pod without any delay (force delete)

12. List "nginx-dev" and "nginx-prod" pod and delete those pods 

13. List all the pods showing name and namespace with a json path expression

14. List all the pods sorted by created timestamp

15. List all the pods sorted by name
