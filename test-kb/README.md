## IAM 設定
$ eksctl utils associate-iam-oidc-provider --region=eu-west-1 --cluster=test-kb --approve
$ eksctl create iamserviceaccount \
  --cluster=test-kb \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::888011373241:policy/yasyam-AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --region eu-west-1 \
  --approve

## ネームスペース用の Fargate profile を作成する
$ eksctl create fargateprofile --cluster test-kb --region eu-west-1 --name fp-kum --namespace kum
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  --set clusterName=test-kb \
  --set region=eu-west-1 \
  --set vpcId=vpc-05012c7d72c261155 \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  -n kube-system

## 起動しないときにトラシュー
$ kubectl logs pod/aws-load-balancer-controller-6cfcc4f747-n2z9l -n kube-system
