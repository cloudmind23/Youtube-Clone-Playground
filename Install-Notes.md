
Deployment of Youtube Clone on Kubernetes cluster with Slack integration

source: https://aakibkhan1.medium.com/deployment-of-youtube-clone-on-kubernetes-cluster-with-slack-integration-bfe621d89b42

Completion steps

Step 1 : Set a base EC2 on AWS

Step 2 : IAM Role for EC2

Step 3 : Configure EC2

You have to install git , docker , jenkins , kubectl etc for docker container and kubernetes deployment

run the following commands

apt update 

vim run.sh

bash run.sh





Step 4 : Setup jenkins pipeline

From the above step everything is installed all we need to setup jenkins ci-cd pipeline

copy the public ip of your ec2 and paste it on browser type →<public_ip>:8080

jenkins login pages comes up. you need admin password
- for this go to your ec2 and connect to it 

run:
sudo su 
cat /var/lib/jenkins/secrets/initialAdminPassword

output is your password and paste it to your jenkins

 Install the suggested plugins
 Setup your jenkins user




Step 5 : ci-cd pipeline to build and push the image to docker hub

Install Plugins listed below in jenkins: 

Terrform
slack notification
Global Slack Notifier
Eclipse Temurin Installer (Install without restart)
NodeJs Plugin (Install Without restart)
Parameterized Trigger (to trigger another pipeline if one completed)
Download all the docker realated plugins

Note → if jenkins is not running on the given website then go to your ec2 security groups and open the 8080 port no
also open port 3000 for the docker container (https://aakibkhan1.medium.com/deployment-of-youtube-clone-on-kubernetes-cluster-with-slack-integration-bfe621d89b42)





step 6 : Setup docker credentials

go to your jenkins →manage jenkins →credentials →global →add credentials

provide your username and password of your dockerhub

id==docker



step 7 : etup tools for jenkins
go to manage jenkins → tools

add jdk
click on add jdk and select installer adoptium.net
choose jdk 17.0.8.1+1version and in name section enter jdk 17


add node js
click on add nodejs
enter node16 in name section
choose version nodejs 16.2.0

add docker →
click on add docker
name==docker
add installer ==download from docker.com

go to manage jenkins →tools →search for terraform
add terraform
provide the name in the name field and untick the install automatically option and give the path /usr/bin/
As terraform is installed in this section it takes it from there
which terraform? /usr/bin/terraform


click on apply and save



Step 8: he Pipeline Script
Go to new item →select pipeline →
in the name section type Youtube Pipeline 1

scroll down to the pipeline script and copy paste the following code

pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Aakibgithuber/deployment-of-youtube.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t youtube-clone ."
                       sh "docker tag youtube-clone aakibkhan1212/youtube-clone:latest "
                       sh "docker push aakibkhan1212/youtube-clone:latest "
                       sh "docker "
                    }
                }
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name youtube-clone -p 3000:3000 aakibkhan1212/youtube-clone:latest'
            }
        }
    }
}


Your application is successfully deployed checkout the dockerhub and to acess your application go to your browser and type
http://your_public_ip:3000



Bonus points:

Step 1 : Integrate Slack notification for pipeline

Integrate Slack notification for pipeline

go to slack.com and make a account
Create a Workspace
Create a new channel
Now you have to browse Slack app directory and search for jenkins
click on add to slack and choose a channel
It will generate token for the jenkins

Now you have to go to Jenkins →manage jenkins →global and scroll down to Slack Notifications
On credentials add the token in the secret text dropdown
Click on Test Connection and you will see Success blink on.


Now again go to your youtube Pipeline and click on configure then add the following code to the pipeline script

pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Aakibgithuber/deployment-of-youtube.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh "docker build -t youtube-clone ."
                        sh "docker tag youtube-clone aakibkhan1212/youtube-clone:latest"
                        sh "docker push aakibkhan1212/youtube-clone:latest"
                    }
                }
            }
        }
        stage('Deploy to Container') {
            steps {
                sh 'docker run -d --name youtube-clone -p 3000:3000 aakibkhan1212/youtube-clone:latest'
            }
        }
    }
    post {
        success {
            slackSend(channel: '#all-the-cloud-hub', color: 'good', message: "Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) was successful.")
        }
        failure {
            slackSend(channel: '#all-the-cloud-hub', color: 'danger', message: "Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) failed.")
        }
        always {
            echo 'Build finished, check Slack for notifications.'
        }
    }
}



It shows success on slack for the pipeline that we created on jenkins

Success



Step 7 : Kuberntes Cluster creation using terraform


Kuberntes Cluster creation using terraform



clone the github repo by
a. mkdir super_mario
b. cd super_mario
c. git clone https://github.com/Aakibgithuber/deployment-of-youtube.git
d. cd deployment-of-youtube/

e. cd EKS-TF
f. edit the backend.tf file by → vim backend.tf (make sure you already have an s3 bucket name)
g. terraform init
h. terraform validate
i. terraform plan
j. terraform apply
k. aws eks update-kubeconfig --name EKS_CLOUD --region us-east-1 (make sure it's your region it created the cluster in) (update the configuration of EKS)





Step 8→ Creation of deployment for EKS

change the directory where deployment and service files are stored use the command → cd ..
create the deployment
kubectl apply -f deployment.yaml

deployment.yaml file is like a set of instructions that tells a computer system, "Hey, here's how you should run and manage a particular application " . It provides the necessary information for a computer system to deploy and manage a specific software application. It includes details like what the application is, how many copies of it should run, and other settings that help the system understand how to keep the application up and running smoothly.
kubectl get all

copy the load balancer ingress and paste it on browser and your application is running there

SUCCESS 



Step 9→ Destroy all the Insrastructure

Don’t forget to destory everything that’s saves of aws bill and you aws account too

cd EKS-TF
terraform destroy --auto-approve
Now go to your EC2 and terminate your Instance
