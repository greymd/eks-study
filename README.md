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
kubectl apply -f frontend-service-lb.yaml -n frontend

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

AWS Load Balancer Controller 使ったこと無いので試してみる
https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html

curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.2.0/docs/install/iam_policy.json
aws iam create-policy \
    --policy-name yasyam-AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json

eksctl utils associate-iam-oidc-provider --region=eu-west-1 --cluster=test-cluster --approve

eksctl create iamserviceaccount \
  --region eu-west-1 \
  --cluster=test-cluster \
  --namespace=frontend \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::888011373241:policy/yasyam-AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve

sudo snap install helm --classic
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master"
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  --set clusterName=test-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  -n frontend

kubectl apply -n frontend -f frontend-svc.yaml
kubectl apply -n frontend -f frontend-ing.yaml


## もし ASG の Desired capacity が 0 になったら
$ eksctl scale nodegroup --region eu-west-1 --cluster test-cluster --name ng-12ddc879 --nodes 2
