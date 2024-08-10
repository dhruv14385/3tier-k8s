# 3 tier application deployment on EKS   
Frontend on ReactJS, Backend on NodeJS and Database on MongoDB. Launch an EC2 instance. Pull repository. Install Docker. Create backend and frontend images and push them to ECR. Create an EKS cluster. Apply database, backend and frontend manifests, thus creating pods for each tier. Create and configure ALB. Install ingress controller for ALB using Helm. Apply ingress manifest. Create a new DNS record for your domain and check that the app is live. 
https://www.youtube.com/live/wgmYbSN6_Is?si=kgeMPADmR56L2GKU 

https://github.com/LondheShubham153/TWSThreeTierAppChallenge 

Create an EC2 instance and name it – 3-tier-hq 

Connect to EC2 

Get updates 
```
sudo apt-get update 
```
Clone repository 
```
git clone https://github.com/LondheShubham153/TWSThreeTierAppChallenge.git 
```
Go to folder TWSThreeTierAppChallenge/frontend 

Install Docker 
```
sudo apt install docker.io 
```
Give user the permission to docker.sock. This is similar to add user to the group. 
```
sudo chown $USER /var/run/docker.sock 
```
Build frontend image. Dockerfile is already in the frontend folder. 
```
docker build –t three-tier-frontend . 
```
Check the images 
```
docker images 
```
Run the image (create container) 
```
docker run -d -p 3000:3000 three-tier-frontend:latest 
```
Open port 3000 to all traffic in SG if required 

Check in the browser if the app is running by going to <EC2-public-IP>:3000	 

Stop the container 
```
docker kill 
```
Go to home directory. Install AWS CLI. You need AWS Access key id and secret access key. 
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" 
sudo apt install unzip 
unzip awscliv2.zip 
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update 
aws configure 
```
 

Prepare to push image to ECR. Create a public repository in ECR. Create separate repository for backend and frontend. Click on ‘View push commands’. Follow the commands. Make sure to build the image in the frontend directory. 

Change directory to backend. Follow above steps and create image for backend and push it to ECR. To create container, use following command. 
```
docker run –d –p 8080:8080 three-tier-backend:latest 
```
Check if container is running.  
```
docker logs <container_name> 
```
You will notice that it is not connected to database because we haven’t created any container for database. 

Install kubectl. Go to home directory and only then install.  
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl 
chmod +x ./kubectl 
sudo mv ./kubectl /usr/local/bin  
kubectl version --short –client 
```
Install eksctl 
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp 
sudo mv /tmp/eksctl /usr/local/bin 
eksctl version 
```
Setup EKS cluster. The command below will take 15 minutes to finish cluster set up. 
```
eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2 
```
Link kubectl to EKS cluster. The command below helps point kubectl to the cluster we want, out of many available clusters. 
```
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster 
```
```
kubectl get nodes 
```
Apply database manifests 

Go to k8s_manifests/mongo directory. Create a new namespace called workshop.  
```
kubectl create namespace workshop 
```
Apply secrets.yaml 
```
kubectl apply –f secrets.yaml 
```
Apply deploy.yaml 
```
kubectl apply –f deploy.yaml 
```
Check deployment 
```
kubectl get deployment –n workshop 
```
Check available pods 
```
kubectl get pods –n workshop 
```
Apply service 
```
kubectl apply –f service.yaml 
```
Check service 

```
kubectl get svc –n workshop 
```
Go to k8s_manifests directory. Edit image path in backend-deployment.yaml to that on ECR 
```
vim backend.yaml 
```
Apply backend deployment and service manifests. 

Check logs to see if backend is connected to MongoDB 

Update image path in frontend deployment manifest. Also update backend URL. 

Apply frontend deployment and service. Then check that all 3 tiers (pods) are running. 

Go to home directory. Download json file for IAM policy for Application Load Balancer. 
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json 
```
Create IAM policy from this json file 
```
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json 
```
Associate this policy with EKS 
```
eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=three-tier-cluster --approve 
```
Create IAM role for EKS. Update your AWS account number and region. 
```
eksctl create iamserviceaccount --cluster=three-tier-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-west-2 
```
Install helm 
```
sudo snap install helm --classic 
```
Add EKS chart repository to helm 
```
helm repo add eks https://aws.github.io/eks-charts 
```
Get update for the repository above 
```
helm repo update eks 
```
 

Check repo list 
```
helm repo list 
```
Install ALB controller 
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller 
```

Check if the controller above is installed  
```
kubectl get deployment -n kube-system aws-load-balancer-controller 
``` 

Change directory to TWSThreeTierAppChallenge/k8s_manifests/ 

Update host in full_stack_lb.yaml and apply full_stack_lb which is an Ingress manifest 

Check ingress 
```
kubectl get ing –n workshop 
```
Create a new CNAME record for your domain in R53 

Use your host name to check if the app is accessible from public internet.
 

 

  

 

 

 
