# Exam1

kubectl get nodes -o json > exam/json-sean.json

# Exam2

kubectl create namespace exam-sean

# Exam3

kubectl create deployment sean-deployment --image=nginx --replicas=2 --namespace=exam-sean

# Exam4

# Exam5
kubectl create deployment nginx-deploy --namespace=exam-sean --replicas=1 --image=nginx:1.16
kubectl set image deployments/nginx-deploy nginx=nginx:1.17

# Exam6
kubectl create -f web-application.yaml

# Exam7