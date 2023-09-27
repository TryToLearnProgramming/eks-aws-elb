# Need to add tags in Cluster Subnets -

	##Public SN -
		kubernetes.io/role/elb			1
		kubernetes.io/cluster/<ClusterName>	shared
	##Private SN -
		kubernetes.io/role/internal-elb		1
		kubernetes.io/cluster/<ClusterName>	shared

## associate IAM OIDC provider with cluster - with out this we can't create iam-service-account

	eksctl utils associate-iam-oidc-provider --region=eu-west-1 --cluster=test-972
	eksctl utils associate-iam-oidc-provider --region=eu-west-1 --cluster=test-972 --approve


## create a service account for the claster

##1 create a policy

	aws iam create-policy --policy-name ALBIngressControllerIAMPolicy --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/master/docs/examples/iam-policy.json
	### if not work then try to cteate it manually, here I created it manually

##2 create and assign a service account with the cluster

	eksctl create iamserviceaccount --cluster=test-972 --namespace=kube-system --name=test-972-aws-load-balancer-service-account --attach-policy-arn=arn:aws:iam::392133049967:policy/AWSLoadBalancerControllerIAMPolicy-Test --override-existing-serviceaccounts --approve

## add aws-loadbalance-ingress
	helm repo add eks https://aws.github.io/eks-charts
	
	helm repo update eks
	
	helm install test-972-aws-elb-controler eks/aws-load-balancer-controller -n kube-system --set clusterName=test-972 --set serviceAccount.create=false --set serviceAccount.name=test-972-aws-load-balancer-service-account --set region=eu-west-1 --set vpcId=vpc-09f746110983de08f --set image.repository=392133049967.dkr.ecr.eu-west-1.amazonaws.com/amazon/aws-load-balancer-controller

### to delete controller - 
	
	helm delete test-972-aws-elb-controler -n kube-system
