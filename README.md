# NodeJs-Project1-CI-CD

This repository contains a project where I’ve managed to create a Build & Deployment for a Node.js app

## Tools Covered
- AWS 
- Git & GitHub 
- Jenkins 
- npm
- Jfrog
- Docker
- Kubernetes 


## Step by Step to Build and Deploy Node.js app
- Step 1 - Create an instance named Jenkins-Server and login to the server. Make sure you open port 8080. Install git, install Java, and Jenkins in /opt, and access Jenkins GUI.
- Step 2 - Create an instance named Build-Server and login to the server. Install git, install Java, npm, docker, eksctl in /opt on build-server
- Step 3 - Master - Slave Configuration 
- Step 4 - Jfrog Account creation. Generate a token in Jfrog and configure the token in Jenkins GUI 
- Step 5 - Install the nodejs plugin, artifactory plugin, Docker pipeline plugin in Jenkins GUI
- Step 6 - Configure nodejs in Global tool Configuration
- Step 7 - Create a dedicated user to use for a Docker login in Jfrog 
- Step 8 - Create a  Continuous Integration pipeline
- Step 9 - Deployment in Kubernetes
- Step 10 - Access Node.Js Application 

## Step 1 - Create an instance named Jenkins-Server and login to the server. Make sure you open port 8080. Install git, install Java, and Jenkins in /opt, and access Jenkins GUI.

- Become root user
```
sudo su - 
```

### Install Git
```
yum install git -y
```
- Note - When your installing 3rd party softwares. It is recommended to use opt directory
```
cd /opt 
```
### Install Java
```
amazon-linux-extras install java-openjdk11 -y
```
- Check Java version 
```
java -version
```
### Install Jenkins 
```
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
```
```
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```
```
yum install jenkins -y
```
```
service jenkins start
```
#### Access the Jenkins Gui 
- Browse - giveurjenkinsserverip:8080 

##### once u access it will ask password. Execute below command in Jenkins-Server 
```
cat /var/lib/jenkins/secrets/initialAdminPassword
```
##### Take password and paste in the Jenkins Gui

- Select (install suggested plugins)

#### Create First Admin User
- Note: I have given random. Give according to your convinience.
- username: admin 
- password: password
- confirm password: password 
- Full name: Pavan
- Email-address: admin@gmail.com  
- Click (save & continue) 
- Click (save & finish)
- Click (start using jenkins)

#### Now start using Jenkins 

## Step 2 - Create an instance named Build-Server and login to the server. Install git, install Java, npm, docker, eksctl in /opt on build-server.

- Become root user
```
sudo su - 
```
### Install Git
```
yum install git -y
```
- Note - When your installing 3rd party softwares. It is recommended to use opt directory
```
cd /opt 
```
### Install Java
```
amazon-linux-extras install java-openjdk11 -y
```
- Check Java version 
```
java -version
```
### Install npm
- Reference - Install npm - https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```
```
. ~/.nvm/nvm.sh
```
```
nvm install 16
```
- To check version 
```
node -e "console.log('Running Node.js ' + process.version)"
```

### Install Docker 
```
yum install docker -y
``` 
```
service docker start
```
```
service docker status
```
- To confirm weather docker service is running or not 
```
docker info  
```
- Docker should have permissions to /var/run/docker.sock

```
chmod 777 /var/run/docker.sock
```
```
cd ~
```
```
exit
```


### Install eksctl as ec2-user. In Master and Slave Configuration, I have given the remote root directory (/home/ec2-user/jenkins).

- Install - AWS CLI 

```
aws --version
```
 - By default you can able to see aws cli as we r using amazon linux ```aws-cli/2.10.0 Python/3.9.11 Linux/5.10.165-143.735.amzn2.x86_64 exe/x86_64.amzn.2 prompt/off```

- Set Up kubectl - https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.23.16/2023-01-30/bin/linux/amd64/kubectl
```
Execution Permissions
```
chmod +x ./kubectl
```
Move kubectl to Usr local bin
```
sudo mv ./kubectl /usr/local/bin
```
Check the kubectl version 
```
kubectl version --short --client
```

- Set Up eksctl - https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
```
```
sudo mv /tmp/eksctl /usr/local/bin
```
```
eksctl version
```
### Create an IAM Role and attach it to EC2 instance

- Go to AWS Management Console 
- Click (Services) 
- Search (IAM) 
- Click (Roles)
- Click (Create role)
##### Select trusted entity
- Select (Aws Service) 
##### Use case 
- Select (EC2) # we are attaching to ec2 
- Click (Next) 
##### Add permissions
##### Search 
- Filter Policies (IAM Full Access) Select (IAM Full Access) (Provides full access to IAM via the AWS Management Console.)
- Filter Policies (Cloud Formation) Select (AWSCloudFormationFullAccess) (Provides full access to AWS CloudFormation.)
- Filter Policies (VPC) Select (AmazonVPCFullAccess) (Provides full access to Amazon VPC via the AWS Management Console.)
- Filter Policies (EC2) Select (AmazonEC2FullAccess) (Provides full access to Amazon EC2 via the AWS Management Console.)
- Filter Policies (Administrator) Select (AdministratorAccess) (Provides full access to AWS services and resources.)
- Click (Next)
##### Role name
- Enter a meaningful name to identify this role (bootstrap_role)
- Click (Create Role)

### Now Attach It To Ec2 Instance

- Go to AWS Management Console 
- Click (Services) 
- Search (EC2) 
- Select (Server) # Where we installed awscli, kubectl
- Select (Actions) 
- Click (Security) 
- Click (Modify IAM Role) 
- Select (bootstrap_role) 
- Click (Update IAM role)


- Create a cluster 
###### I given name as mycluster. You can give any name.
#####  Execute below command 

```
eksctl create cluster --name mycluster \
                      --region us-east-1 \
                      --node-type t2.small \
```
###### It will take 15 minutes time to execute. 
###### Meanwhile if u go to AWS Management Console -> Services -> CloudFormation  (u can see # eksctl-mycluster-cluster - CREATE_IN_PROGRESS)
###### Once its executed successfully u can able to see (eksctl-mycluster-cluster - CREATE_COMPLETE)
###### go to AWS Management Console -> Services -> Elastic Kubernetes Service -> you can see (mycluster)  
###### new instances with t2.small is created 

Validate cluster by checking the nodes
```
kubectl get nodes
```

### Step 3 - Master - Slave Configuration 

- Note - I am not running jobs on the Jenkins-Master server. I am running jobs on Build-Server.
##### Add Slave (Build-Server) credentials
- Click (Manage Jenkins) 
- Click (Manage Credentials) 
###### Under Stores scoped to Jenkins
- Click (global)
###### Global credentials (unrestricted)
- Click (+ Add Credentials) 
- Kind (SSH username with private key)
- Scope (Global(jenkins,nodes,items,all child items, etc)
- id -  Jenkins-slave
- Username - ec2-user
##### Private key 
- Click (Enter directly)
- Click (ADD) 
- (Give private key of Instance which is .pem file) 
- Click (create) 

### Add Build-Server as a permanent slave to Jenkins-Server  

- Click (Manage Jenkins)
- Click (Manage Nodes and Clouds) 
- Click (New Node) 
- Node name - Jenkins-slave
- Click (Permanent Agent) 
- Number of executors (2)
- Remote root directory (/home/ec2-user/jenkins)
- ***Lables (nodeslave) # This should be given in the pipeline

##### Launch method
- (Launch agent via ssh) 
- Usage (only build jobs with label expressions matching this node) 
- Host (Private Ip of Build-Server) 
- Credentials - (ec2-user)
- Host Key Verification Strategy - (Non verifyng Verification strategy) 
- Click (save) 

##### Now write the Declarative Pipeline Script, and the code looks like the below:

```
pipeline{
    agent{
        node{
            label "nodeslave"
        }
    }
}
```



### Step 4 - Jfrog Account Creation (One Time procedure)

- Search - https://jfrog.com
- Select (products - JFROG Artifactory) 
- Select (Free Trial)
- Select (Skip Trial and Start with a Free JFrog Instance)
- Select (Sign Up with Github) 
- Select (Authorize GH-prod-SSO)

- Create a Hostname* - (give any name) 
- Give Your First Name, Last Name
- Hosting Preferences (aws)
- Click (Try It Now) 

##### It will ask for Sign in 
- Select (github) 
- Select (Authorize GH-prod-SSO)

- Select (Lets Go)
- What is the purpose of this JFrog account? (Personal)
- Seect Your Role (DevOps Engineer) 
- What are you interested in? (CI/CD) # Select your choices

- Note - Next time onwards u login with github when it asks signin

### Generate token

- Select (User menu) # Your profile name
- Select (New user) 
- Select (Access token) # under user management
- Select (+ Generate token) 

##### Generate Token 
- Select - (Scoped Token)
- Description - (Demo-token) # Give any meaningful name 
- Token Scope - (Admin) # As per your specification
- User name - (Your mail id which is connected with github account) 
- Expiration time - # Depends on you - Example: Never
- Click (Generate) 

###### You will get a token. Copy that token. Copy the generated token into Jenkins. Go to Jenkins Gui

- Click (Manage Jenkins) 
- Click (Manage Credentials) 
##### Under Stores scoped to Jenkins
- Click (System) 
- Click (Global credentials (unrestricted))
- Click (Add Credentials) 
###### kind (Username with password) 
- username (your mail id which u connected with github) 
- Password (Generated token from jfrog paste here) 
- ID - Jfrog-token  #ID is must. Do remember because we give this in pipeline
- Click (Create) 




### Step 5 - Install the nodejs plugin, artifactory plugin, Docker pipeline plugin in Jenkins GUI.

##### Install nodejs plugin 

- Click (Manage Jenkins)
- Click (install plugins)
- Click (Available Plugins) - Search ('NodeJs') 
- Click ('NodeJs') 
- Click (Install without Restart)

##### Install Docker pipeline plugin

- Click (Manage Jenkins) 
- Click (Manage Plugins) 
- Select (Available Plugins) 
- Search (Docker Pipeline) 
- Select (Docker Pipeline)
- Select (Install without restart) 


##### Install Artifactary Plugin

- Click (Manage Jenkins) 
- Click (Manage Plugins) 
- Select (Available Plugins) 
- Search (Artifactory) 
- Select (Artifactory)
- Select (Install without restart) 

### Step 6 - Configure nodejs in Global tool Configuration

- Click (Dashboard)
- Click (Manage Jenkins) 
- Click (Global tool configuration)
###### Under NodeJs
- Click (Add Nodejs) 
- Name (node-js) #Same name should be given the pipeline code - tools {nodejs "node-js"}
- Version (NodeJs-16.6.0)
- Select (Install automatically)
- Click (Apply)
- Click (Save)

## Write a declarative pipeline for each stage.
- Note - Fork the repository then change wherever i mentioned
#### Stage 1 - Clone the code from github repository 
```
pipeline {
    agent{
        node {
            label "nodeslave"
        }
    }
    stages{
        stage ("clone the code"){
            steps{
                git branch: 'main', url: 'https://github.com/Bathalapalli-SaiRangaPavan/NodeJs-Project1-CI-CD.git'#https://github.com/<give your name>/NodeJs-Project1-CI-CD.git
            }
        }
    }
}
```

##### Stage 2 - Build the code
```
pipeline {
    agent{
        node {
            label "nodeslave"
        }
    }
    tools {nodejs "node-js"}
    stages{
        stage ("clone the code"){
            steps{
                git branch: 'main', url: 'https://github.com/Bathalapalli-SaiRangaPavan/NodeJs-Project1-CI-CD.git' #https://github.com/<give your name>/NodeJs-Project1-CI-CD.git
            }
        }
       
        stage ("Build the code"){
            steps{
                sh "npm install"
            }
        }
    }
}
```

##### Stage 3 - Unit testing 

```
pipeline {
    agent{
        node {
            label "nodeslave"
        }
    }
    tools {nodejs "node-js"}
    stages{
        stage ("clone the code"){
            steps{
                git branch: 'main', url: 'https://github.com/Bathalapalli-SaiRangaPavan/NodeJs-Project1-CI-CD.git' #https://github.com/<give your name>/NodeJs-Project1-CI-CD.git
            }
        }
       
        stage ("Build the code"){
            steps{
                sh "npm install"
            }
        }

        stage ("Unit test"){
            steps{
                sh "npm test"
            }
        }
    }
}
```


###### Create a Dockerfile 

```
FROM node
WORKDIR /app
ADD . /app
RUN npm install
EXPOSE 3000
CMD npm start
```


##### Create a Docker Artifactory in Jfrog 

###### Once u login to Jfrog u can able to see left side (Artifactory, Xray, Distribution, Pipelines, Integrations) 
- Select (Artifactory)
- Select (Quick SetUp)
- Now You can able to see Available package types (docker)
- Repositories prefix  (nodebuild)
- Click (Create)

- Click on (Artifactory) # U can able to (quick setup, Repositories, Packages, Artifacts, Build) 
- Select (Artifacts) 
###### Now u can able to see nodebuild-docker, nodebuild-docker-local, nodebuild-docker-remote, nodebuild-docker-remote-cache
###### when u click on nodebuild-docker-local u cannot able to see any folder 


##### Stage 4 - Docker build & Docker Publish 
```
def registry = "https://bsairangapavan.jfrog.io/" ### "https://your.jfrog.io
def imageName = "bsairangapavan.jfrog.io/nodebuild-docker/newimageversion" ### "your.jfrog.io/yourrepositoryprfix/yourpreferredimagename
def version   = "1.0.0"    ### Image version i defined 1.0.0
pipeline {
    agent{
        node {
            label "nodeslave"
        }
    }
    tools {nodejs "node-js"}
    stages{
        stage ("clone the code"){
            steps{
                git branch: 'main', url: 'https://github.com/Bathalapalli-SaiRangaPavan/NodeJs-Project1-CI-CD.git'   #https://github.com/<give your name>/NodeJs-Project1-CI-CD.git
            }
        }
       
        stage ("Build the code"){
            steps{
                sh "npm install"
            }
        }

        stage ("Unit test"){
            steps{
                sh "npm test"
            }
        }
        stage(" Docker Build ") {
            steps {
                script {
                    app = docker.build(imageName+":"+version)
                }
            }
        }
        stage (" Docker Publish "){
            steps {
                script { 
                   docker.withRegistry(registry, 'Jfrog-token'){              #### I given Jfrog-token. Instead whaterver you defined give here.
                      app.push()
                    }    
                }
            }
        }  
    }
}
```
###### when u click on nodebuild-docker-local u can able to see 1.0.0

### Step 7 - Create a dedicated user to use for a Docker login in Jfrog 

- Login to your Jfrog Account
- Select (User Menu)  #Your profile name
- Click (New User) 
- username (dockerjfrog) # give anyname
- Email Address (give your email address) 
- Select (Administrater Platform) 
- Password (give your password) 
- Confirm Password (give your password)
- Click (save) 


###### Go to Build server and login into your jfrog account. 
```
sudo docker login bsairangapavan.jfrog.io  # login (give your.jfrog.io)
```
###### it will ask credentials 
```
username - dockerjfrog # (give your email address) 
```
```
password - (give your password) 
```
##### Login succeeded
##### whenever u login to jfrog repository the credentials will be stored in the /root/.docker/config.json 
```
sudo cat /root/.docker/config.json 
```
##### Check images
```
sudo docker images 
```

### Step 8 - Create a  Continuous Integration pipeline
##### Login Jenkins Gui
- Click (New Item) 
- Click (Pipeline)
- Name (nodejs-ci)
- Click (Ok)
##### Under Pipeline - Defination
- Select (Pipelinescript from SCM)
- SCM (git) 
- Repository URL - (https://github.com/Bathalapalli-SaiRangaPavan/NodeJs-Project1-CI-CD.git)  
- Credentials (None) # As its a public repository 
##### Branches to build
- Branch Specifier - (*/main)
- Click (Apply) 
- Click (Save) 
- Click (Build Now)


![nodeci](https://user-images.githubusercontent.com/121741348/224545297-9b21d0d9-9245-49e3-9101-9aabd52babc2.png)


##### Check images in Build-server
```
sudo docker images 
```
![image](https://user-images.githubusercontent.com/121741348/224545559-bebc40d4-de1d-4e19-83ee-847ff822c55b.png)


### Step 8 - Deployment in kubernetes 

- Execute below command in Build Server using root as i installed docker in /opt 
```
cat ~/.docker/config.json | base64 -w0
```
- copy and paste in the secret.yaml - dockerconfigjson


##### Create secret.yaml 

```
apiVersion: v1
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSJic2FpcmFuZ2FwYXZhbi5qZnJvZy5pbyI6IHsKCQkJImF1dGgiOiAiWkc5amEyVnlhbVp5YjJjNlFXUnRhVzVBTVRJek5EVTIiCgkJfQoJfQp9
kind: Secret
metadata:
  name: dockerjfrog
type: kubernetes.io/dockerconfigjson
```

##### Create Deployment.yaml 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-cicd-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs-cicd
  template:
    metadata:
      labels:
        app: nodejs-cicd
    spec:
      imagePullSecrets:
      - name: dockerjfrog 
      containers:
      - name:  nodejs-cicd
        image: bsairangapavan.jfrog.io/nodebuild-docker/newimageversion:1.0.0
        imagePullPolicy: Always
        ports:
        - containerPort: 3000
```

### Create Service.yaml 

```
apiVersion: v1
kind: Service
metadata:
  name: nodejs-cicd-service
spec:
  type: NodePort
  selector:
    app: nodejs-cicd
  ports:
  - nodePort: 30086
    port: 3000
    targetPort: 3000 
 ```
 
 ### Create deploy.sh 
 
 ```
 kubectl apply -f service.yaml 
 kubectl apply -f deployment.yaml 
 kubectl apply -f service.yaml 
 ```
 ##### Give permission 
 ```
 chmod 777 deploy.sh
 ```
 ```
 git update-index --chmod=+x deploy.sh
 ```
 ##### Stage 5 - Deployment 
 
 ```
 def registry = "https://bsairangapavan.jfrog.io/"
def imageName = "bsairangapavan.jfrog.io/nodebuild-docker/newimageversion" 
def version   = "1.0.0"
pipeline {
    agent{
        node {
            label "nodeslave"
        }
    }
    tools {nodejs "node-js"}
    stages{
        stage ("clone the code"){
            steps{
                git branch: 'main', url: 'https://github.com/Bathalapalli-SaiRangaPavan/NodeJs-Project1-CI-CD.git'
            }
        }
       
        stage ("Build the code"){
            steps{
                sh "npm install"
            }
        }

        stage ("Unit test"){
            steps{
                sh "npm test"
            }
        }
        stage(" Docker Build ") {
            steps {
                script {
                    app = docker.build(imageName+":"+version)
                }
            }
        }
        stage(" Docker Publish "){
            steps {
                script { 
                   docker.withRegistry(registry, 'Jfrog-token'){
                      app.push()
                    }    
                }
            }
            
        }  
        stage("Deployment"){
            steps {
                sh "./deploy.sh"
            }
        }
    }
}
```

![ci-cd](https://user-images.githubusercontent.com/121741348/224558955-04015aab-38ac-46f8-99fc-805f4d1bfcc5.png)

##### Execute kubectl get all in build-server 

![all](https://user-images.githubusercontent.com/121741348/224559106-5f8905c9-a1d6-4bc2-a670-34f5f53e28d4.png)

- Make sure u open security group - port 30086 in any node 
### Step 10 - Access Node.Js Application 
- Browse - eksctlgeneratednodeip:30086 - when you do u can able to access below page

![output](https://user-images.githubusercontent.com/121741348/224559363-6846d63d-4e10-44c2-9289-85ffecdc05eb.png)



