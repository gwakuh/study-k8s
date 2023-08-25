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

9. Create the nginx pod with version 1.17.4 and expose it on port 80
```sh
$ kubectl run nginx --image=nginx:1.17.4 --port=80
```

8. Change the Image version to 1.15-alpine for the pod you just created and verify the image version is updated.
```sh
$ kubectl set image pod/nginx nginx=nginx:1.15-alpine

$ kubecdtl describe pod nginx
Name:             nginx
Namespace:        exam-sean
Priority:         0
Service Account:  default
Node:             gke-oscar-cluster-2-default-pool-c68e7d97-44o9/10.128.15.228
Start Time:       Fri, 25 Aug 2023 12:11:07 +0900
Labels:           run=nginx
Annotations:      <none>
Status:           Running
IP:               10.32.8.149
IPs:
  IP:  10.32.8.149
Containers:
  nginx:
    Container ID:   containerd://6af63b6a88cc747a222f866a93d969130acabaca298c5dad8a01b61155cae497
    Image:          nginx:1.15-alpine
    Image ID:       docker.io/library/nginx@sha256:57a226fb6ab6823027c0704a9346a890ffb0cacde06bc19bbc234c8720673555
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 25 Aug 2023 12:14:03 +0900
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 25 Aug 2023 12:11:09 +0900
      Finished:     Fri, 25 Aug 2023 12:14:02 +0900
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xpbzt (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-xpbzt:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age                  From               Message
  ----    ------     ----                 ----               -------
  Normal  Scheduled  3m11s                default-scheduler  Successfully assigned exam-sean/nginx to gke-oscar-cluster-2-default-pool-c68e7d97-44o9
  Normal  Pulled     3m10s                kubelet            Container image "nginx:1.17.4" already present on machine
  Normal  Killing    16s                  kubelet            Container nginx definition changed, will be restarted
  Normal  Pulling    16s                  kubelet            Pulling image "nginx:1.15-alpine"
  Normal  Created    15s (x2 over 3m10s)  kubelet            Created container nginx
  Normal  Started    15s (x2 over 3m9s)   kubelet            Started container nginx
  Normal  Pulled     15s                  kubelet            Successfully pulled image "nginx:1.15-alpine" in 1.301554823s (1.301563528s including waiting)

$
```

7. Change the Image version back to 1.17.1 for the pod you just updated and observe the changes
```sh
$ kubectl set image pod/nginx nginx=nginx:1.17.1
$ kubectl describe pod nginx
Name:             nginx
Namespace:        exam-sean
Priority:         0
Service Account:  default
Node:             gke-oscar-cluster-2-default-pool-c68e7d97-44o9/10.128.15.228
Start Time:       Fri, 25 Aug 2023 12:11:07 +0900
Labels:           run=nginx
Annotations:      <none>
Status:           Running
IP:               10.32.8.149
IPs:
  IP:  10.32.8.149
Containers:
  nginx:
    Container ID:   containerd://22f7e2c6244843bca972eb556c375fe2871beb2d2b8acb84d36ec28e8eb0e0e3
    Image:          nginx:1.17.1
    Image ID:       docker.io/library/nginx@sha256:b4b9b3eee194703fc2fa8afa5b7510c77ae70cfba567af1376a573a967c03dbb
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Fri, 25 Aug 2023 12:15:48 +0900
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 25 Aug 2023 12:14:03 +0900
      Finished:     Fri, 25 Aug 2023 12:15:43 +0900
    Ready:          True
    Restart Count:  2
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xpbzt (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-xpbzt:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age                  From               Message
  ----    ------     ----                 ----               -------
  Normal  Scheduled  4m57s                default-scheduler  Successfully assigned exam-sean/nginx to gke-oscar-cluster-2-default-pool-c68e7d97-44o9
  Normal  Pulled     4m56s                kubelet            Container image "nginx:1.17.4" already present on machine
  Normal  Pulling    2m2s                 kubelet            Pulling image "nginx:1.15-alpine"
  Normal  Pulled     2m1s                 kubelet            Successfully pulled image "nginx:1.15-alpine" in 1.301554823s (1.301563528s including waiting)
  Normal  Killing    21s (x2 over 2m2s)   kubelet            Container nginx definition changed, will be restarted
  Normal  Pulling    21s                  kubelet            Pulling image "nginx:1.17.1"
  Normal  Created    16s (x3 over 4m56s)  kubelet            Created container nginx
  Normal  Started    16s (x3 over 4m55s)  kubelet            Started container nginx
  Normal  Pulled     16s                  kubelet            Successfully pulled image "nginx:1.17.1" in 4.426598992s (4.426608203s including waiting)
```

10. Create a redis pod and expose it on port 6379
```sh
$ kubectl run redis --image=redis --port=6379
```

11. Delete the pod without any delay (force delete)
```sh
$ kubectl delete pod redis --force
```

12. List "nginx-dev" and "nginx-prod" pod and delete those pods
```sh
$ kubectl get pods/nginx-dev pods/nginx-prod -o wide
$ kubectl delete pod nginx-dev nginx-prod
```

13. List all the pods showing name and namespace with a json path expression
```sh
$ kubectl get pods -o=jsonpath="{range .items[*]}[{.metadata.name}, {.metadata.namespace}]{'\n'}{end}"
```

14. List all the pods sorted by created timestamp
```sh
$ kubectl get pods --sort-by=".metadata.creationTimestamp"
```

15. List all the pods sorted by name
```sh
$ kubectl get pods --sort-by=".metadata.name
```
