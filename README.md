eksctl create cluster --name test-cluster --region eu-west-1 --fargate
kubectl get pods --all-namespaces -o wide
kubectl create namespace frontend
kubectl apply -f ./frontend-deployment.yaml -n frontend

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  selector:
    matchLabels:
      app: frontend
  replicas: 2
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: nginx:1.19
        imagePullPolicy: Always
        ports:
        - containerPort: 80
        env:
        - name: TEST
          value: test
```

kubectl get deployment -n frontend
kubectl get pod -n frontend

```
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```
