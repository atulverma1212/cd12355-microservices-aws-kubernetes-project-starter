
### Install Kubectl: 

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
aws sts get-caller-identity

```



### Steps:

* Create IAM user with Administrative Access
*  Create EKS cluster using following command: 
```
eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name my-nodes --node-type t3.small --nodes 1 --nodes-min 1 --nodes-max 2
```

* The following commands need to run evertime the console restarts: 
```
git config --global user.email "verma@gmail.com"
git config --global user.name "atul.verma"
```

* **Give administrative access if you face the following error**: 
Error: getting availability zones: error getting availability zones for region us-east-1: operation error EC2: DescribeAvailabilityZones, https response error StatusCode: 403, RequestID: c0769ecb-eaec-49e8-9d6f-e92b88791b08, api error UnauthorizedOperation: You are not authorized to perform this operation. User: arn:aws:iam::544060223332:user/admin is not authorized to perform: ec2:DescribeAvailabilityZones because no identity-based policy allows the ec2:DescribeAvailabilityZones action

* Update the Kubeconfig on local machine
```
aws eks --region us-east-1 update-kubeconfig --name my-cluster
```

* Output of the above command: 
```
Added new context arn:aws:eks:us-east-1:544060223332:cluster/my-cluster to /root/.kube/config
```

* View the entire Kubeconfig file on local machine 
```
kubectl config view
```

* Delete EKS cluster command 
```
eksctl delete cluster --name my-cluster --region us-east-1
```

* Create psql database in k8s manually using the provided files and following commands. Also, you will need to apply port forwarding in order to connect to postgresql database.: 
```
kubectl apply -f pvc.yaml
kubectl apply -f pv.yaml
kubectl apply -f postgresql-deployment.yaml
kubectl get pods
kubectl exec -it postgresql-6889d46b98-bd84h -- bash
psql -U myuser -d mydatabase
\l 
\q 
kubectl apply -f postgresql-service.yaml
# List the services
kubectl get svc

# Set up port-forwarding to `postgresql-service`
kubectl port-forward service/postgresql-service 5433:5432 &
```


### Run seed files 

* install psql client 
```
apt update
apt install postgresql postgresql-contrib
```

* Run seed files & Checking the tables: 
```
export DB_PASSWORD=mypassword
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U myuser -d mydatabase -p 5433 < 1_create_tables.sql
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U myuser -d mydatabase -p 5433 < 2_seed_users.sql
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U myuser -d mydatabase -p 5433 < 3_seed_tokens.sql
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U myuser -d mydatabase -p 5433
select *from users;
\q
```

* Closing the forwarded ports
```
ps aux | grep 'kubectl port-forward' | grep -v grep | awk '{print $2}' | xargs -r kill
```

## Deploy the Analytics Application
* Create Dockerfile 
* Create ECR repository and cloudbuild project. Set environment variables in it
* Get AWS Account id using following command: 
```
aws sts get-caller-identity --query Account --output text
```
* Add ECR permission to IAM role created by codebuild. ADd: AmazonEC2ContainerRegistryPowerUser

## Important k8s commands: 

* Get commands: 
```
kubectl get secret <NAME OF THE Secret> -o jsonpath="{.data.DB_PASSWORD}" | base64 -d
kubectl get pods 
kubetctl get svc
kubectl get deployments
```

* Other commands: 
```

kubectl delete pod <pod_name> -n <namespace>
kubectl exec -it <pod_name> -- bash
kubectl describe pod coworking-555f8b9498-6p9qt
kubectl logs coworking-555f8b9498-6p9qt
```

### To test the deployment:

curl aa0f3c19be8cb4e2cbeea42366138778-1393511268.us-east-1.elb.amazonaws.com:5153/api/reports/daily_usage

### Cloudwatch Integration: 

* Attach CloudwatchAgentServerPolicy. Replace my-worker-node-role with your EKS cluster's Node Group's IAM role.
```
aws iam attach-role-policy \
--role-name my-worker-node-role \
--policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy 

aws eks create-addon --addon-name amazon-cloudwatch-observability --cluster-name my-cluster

```