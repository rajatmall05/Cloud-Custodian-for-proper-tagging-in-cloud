
# CloudCustodian set as cronjob in minikube 

Cloud Custodian is a rules engine for managing public cloud accounts and resources. It allows users to define policies to enable a well managed cloud infrastructure, that's both secure and cost optimized .


## setting-up minikube on ubuntu
Minikube is a lightweight Kubernetes implementation that creates a VM on your local machine and deploys a simple cluster containing only one node.


### set-up 
1. sudo apt-get update -y && sudo apt-get upgrade -y
2. sudo apt install virtualbox virtualbox-ext-pack -y
3. wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
4. sudo cp minikube-linux-amd64 /usr/local/bin/minikube
5. sudo chmod 755 /usr/local/bin/minikube
6. minikube version
7. curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
8. chmod +x ./kubectl
9. sudo mv ./kubectl /usr/local/bin/kubectl
10. kubectl version -o json
11. minikube start

#### note : make sure to enable virtualization on your system
## setting up directory in local environment
Create two file 

1. mailer.yml
2. policy.yml

## In mailer.yml :
1. Go AWS console, create a new standard SQS queue (quick create is fine). Copy the queue URL to queue_url in mailer.yml
2. In AWS, locate or create a role that has read access to the queue. Grab the role ARN and set it as role in mailer.yml
3. Make sure your email address is verified in SES, and set it as from_address in mailer.yml

This will look something like this :

![Screenshot from 2023-05-19 10-29-16](https://github.com/rajatmall05/Cloud-Custodian/assets/126334005/12d6a0e6-21c5-4f29-853f-d1a65f2c67ff](https://user-images.githubusercontent.com/126334005/239445518-12d6a0e6-21c5-4f29-853f-d1a65f2c67ff.png)

![Screenshot from 2023-05-19 12-03-39](https://github.com/rajatmall05/Cloud-Custodian/assets/126334005/59d11f73-f68d-4313-8bf8-a5646f545514)


## in policy.yml :

![Screenshot from 2023-05-19 10-54-06](https://github.com/rajatmall05/Cloud-Custodian/assets/126334005/d4744617-9528-4d15-8c5d-97e666b4f841)

## creating Dockerfile with base image of cloudcustodian/c7n

![Screenshot from 2023-06-02 17-31-50](https://github.com/rajatmall05/Clous-Custodian-minkube/assets/126334005/3cf2e3f4-1bf1-4c65-9f85-f382955d4c13)


#### note :
1. This "CMD" command is taken from cloudcustodian/c7n . 
2. While creating DockerFile i have coppied custom template for displaying content in better form while recieving in mail , you can check custom.html.j2 file in the repository

## Pushing Docker file to ECR
1. Create a public repo in ECR
2. Follow the steps in ECR to push the Dockerfile 
3. In this stage you will also build your image in your local 
4. Copy the URL of your image form your public repo

## creating cronjob in minikube

![Screenshot from 2023-05-19 11-50-21](https://github.com/rajatmall05/Cloud-Custodian/assets/126334005/0dd6da04-f0b1-4a10-9924-9cf2b98cb054)

*save this file as cronjob.yml

#### note-1 : in this I have passed region , access-key and secret-access-key under environment variable this can also be done through kube2iam which helps in passing aws roles in k8s . 

#### note-2 : in "args" we specified run command of cloud-custodian.

#### note-3 : in this cron-job i have pulled the image which i have uploaded in ECR .
## running the cronjob 
This cron job will run in every 5 min so , it will check the tags of the policy that you created and if it finds any unidentified tags so it will send email of that missing tag.

#### running commands to run crojob
1. kubectl apply -f cronjob.yml 
2. kubectl get pods -a
3. kubectl logs -f <pod-name>

#### you can also check cronjob in minikube dashboard

minikube dashboard > cronjob
    
#### note : While watching pods in minkube , it will always show pods going in running state and eventually goes down , this is normal because cloud-custodian does it job and exit the script . 

#### For this we have used cronjob so when the pod goes down , the cron job triggers itself and it will again run the pod . 
![Screenshot from 2023-05-19 12-44-13](https://github.com/rajatmall05/Clous-Custodian-minkube/assets/126334005/4df51642-f03d-46a2-8f4c-7a511f150313)

### Using k8s secrets for passing aws-credentials 
1. Create secret.yml 
    
![image](https://github.com/rajatmall05/Clous-Custodian-minkube/assets/126334005/513c0bd1-602d-416b-8f1e-22a6e00754b6)
    
2. Passing secrets in cron-job 
    
![Screenshot from 2023-06-02 17-25-30](https://github.com/rajatmall05/Clous-Custodian-minkube/assets/126334005/3020e2f5-1b06-47a3-8c82-81e7c1f82d76)

#### note : now follow the same steps (running a cron-job) that are mentioned above .
