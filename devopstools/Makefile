create-secret-generator:
	aws --region us-east-1 cloudformation create-stack \
		--capabilities CAPABILITY_IAM \
		--stack-name cfn-secret-provider \
		--template-body file://~/yurtah/devopstools/cloudformaiton/cfn-resource-provider.yaml
	aws cloudformation wait stack-create-complete  --stack-name cfn-secret-provider

delete-secret-generator:
	aws cloudformation delete-stack --stack-name cfn-secret-provider


create-ssh-keys:
	aws cloudformation create-stack --stack-name eks-node-secrets --template-body file://~/yurtah/devopstools/cloudformaiton/ec2-sshkeypair.yaml


delete-ssh-keys:
	aws cloudformation delete-stack --stack-name eks-node-secrest

get-private-key:
	aws ssm get-parameter --name /eks-node-secrets/private-key --with-decryption --output text --query 'Parameter.Value' > ~/.ssh/eks-node-secrets.pem
	chmod 600 ~/.ssh/eks-node-secrets.pem

create-eks:
	aws cloudformation create-stack --stack-name eks-dev \
		--template-body file://~/yurtah/devopstools/cloudformaiton/eks.yaml \
		--capabilities CAPABILITY_IAM \
		--parameter ParameterKey=VpcBlock,ParameterValue=10.20.0.0/16 \
			ParameterKey=ClusterVersion,ParameterValue=1.15 \
			ParameterKey=Subnet01Block,ParameterValue=10.20.64.0/18 \
			ParameterKey=Subnet02Block,ParameterValue=10.20.128.0/18 \
			ParameterKey=Subnet03Block,ParameterValue=10.20.192.0/18 \
			ParameterKey=Subnet01Az,ParameterValue=0 \
			ParameterKey=Subnet02Az,ParameterValue=1 \
			ParameterKey=Subnet03Az,ParameterValue=2

delete-eks:
	aws cloudformation delete-stack --stack-name eks-dev

create-workers:
	aws cloudformation create-stack \
		--capabilities CAPABILITY_IAM \
		--stack-name eks-dev-workers \
		--template-body file://~/yurtah/devopstools/cloudformaiton/amazon-eks-nodegroup.yaml \
		--parameter ParameterKey=ClusterControlPlaneSecurityGroup,ParameterValue=sg-0b3579442273d65ab \
			ParameterKey=ClusterName,ParameterValue=eks-dev \
			ParameterKey=KeyName,ParameterValue=eks-node-secrets-keypair \
			ParameterKey=NodeAutoScalingGroupDesiredCapacity,ParameterValue=2 \
			ParameterKey=NodeAutoScalingGroupMaxSize,ParameterValue=5 \
			ParameterKey=NodeAutoScalingGroupMinSize,ParameterValue=1 \
			ParameterKey=NodeGroupName,ParameterValue=eks-dev-workers \
			ParameterKey=NodeImageId,ParameterValue=ami-06d4f570358b1b626 \
			ParameterKey=NodeInstanceType,ParameterValue=t2.micro \
			ParameterKey=NodeVolumeSize,ParameterValue=8 \
			ParameterKey=Subnets,ParameterValue=subnet-052d03dae1e7d7043\\,subnet-011ff53c08f0004c9\\,subnet-054f2dc9a857e1136 \
			ParameterKey=VpcId,ParameterValue=vpc-0e8da461bbe4714b6 
	aws cloudformation wait stack-create-complete --stack-name eks-dev-workers
	kubectl apply -f cloudformaiton/aws-auth-cm.yaml

update-workers:
	aws cloudformation update-stack \
		--capabilities CAPABILITY_IAM \
		--stack-name eks-dev-workers \
		--template-body file://~/yurtah/devopstools/cloudformaiton/amazon-eks-nodegroup.yaml \
		--parameter ParameterKey=ClusterControlPlaneSecurityGroup,ParameterValue=sg-0b3579442273d65ab \
			ParameterKey=ClusterName,ParameterValue=eks-dev \
			ParameterKey=KeyName,ParameterValue=eks-node-secrets-keypair \
			ParameterKey=NodeAutoScalingGroupDesiredCapacity,ParameterValue=2 \
			ParameterKey=NodeAutoScalingGroupMaxSize,ParameterValue=10 \
			ParameterKey=NodeAutoScalingGroupMinSize,ParameterValue=1 \
			ParameterKey=NodeGroupName,ParameterValue=eks-dev-workers \
			ParameterKey=NodeImageId,ParameterValue=ami-06d4f570358b1b626 \
			ParameterKey=NodeInstanceType,ParameterValue=t2.micro \
			ParameterKey=NodeVolumeSize,ParameterValue=8 \
			ParameterKey=Subnets,ParameterValue=subnet-052d03dae1e7d7043\\,subnet-011ff53c08f0004c9\\,subnet-054f2dc9a857e1136 \
			ParameterKey=VpcId,ParameterValue=vpc-0e8da461bbe4714b6

delete-workers:
	aws cloudformation delete-stack --stack-name eks-dev-workers

create-db-vpc:
	aws cloudformation create-stack \
		--stack-name eks-dev-db-vpc \
		--template-body file://~/yurtah/devopstools/cloudformaiton/db-vpc.yaml \
		--parameter ParameterKey=VpcBlock,ParameterValue=10.30.0.0/16 \
			ParameterKey=Subnet01Block,ParameterValue=10.30.64.0/18 \
			ParameterKey=Subnet02Block,ParameterValue=10.30.128.0/18 \
			ParameterKey=Subnet03Block,ParameterValue=10.30.192.0/18

delete-db-vpc:
	aws cloudformation delete-stack --stack-name eks-dev-db-vpc

create-db:
	aws cloudformation create-stack \
		--stack-name db-dev \
		--template-body file://~/yurtah/devopstools/cloudformaiton/db.yaml \
		--parameter ParameterKey=ParentVPCStack,ParameterValue=eks-dev-db-vpc \
			ParameterKey=ParentSecurityGroupIds,ParameterValue=sg-0e5e390773abe12ba \
			ParameterKey=DBInstanceID,ParameterValue=db-dev \
			ParameterKey=DBName,ParameterValue=mydatabase \
			ParameterKey=DBUsername,ParameterValue=root \
			ParameterKey=Application,ParameterValue=yurtahapp \
			ParameterKey=ServiceOwnersEmailContact,ParameterValue=yurtahconstruction@gmail.com \
			ParameterKey=Confidentiality,ParameterValue=private

delete-db: 
	aws cloudformation delete-stack --stack-name db-dev

create-peering:
	bin/aws-vpc-create-peering-connection.sh vpc-0e8da461bbe4714b6 vpc-0f700c608c0ddadab cloudformaiton/eks-dev-db.yaml

delete-peering:
	aws cloudformation delete-stack --stack-name eks-dev-db-peering

