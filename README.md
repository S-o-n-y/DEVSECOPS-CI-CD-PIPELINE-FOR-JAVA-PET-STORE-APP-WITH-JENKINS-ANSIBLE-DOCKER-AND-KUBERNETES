# DEVSECOPS-CI-CD-PIPELINE-FOR-JAVA-PET-STORE-APP-WITH-JENKINS-ANSIBLE-DOCKER-AND-KUBERNETES
project
DEVSECOPS: (short for development, security, and operation) is a development practice that integrates security initiatives at every stage of the software development lifecycle to deliver robust and secure applications.

•	From the above fig it shows the DevSecOps CI/CD pipeline for pet store app, firstly when developer push the code into GitHub on that time Jenkins trigger and clone the updated code.
•	Next Jenkins going to compile the code with maven build tool and it going to test the code and next the code is tested by the SonarQube for code quality (or) vulnerability, code smells and bugs.
•	Next build the application code with the help of maven build tool and after build we got a .war file (this is  java-based application) next we are going to use another security check OWSAP dependency-check it is going to check vulnerability of each dependency.
•	Next, we are integrating the ansible with the Jenkins and with the help of ansible playbooks we are going to create docker images and tags and push the tagged images into docker hub with the help of ansible playbooks.
•	After creating the image, we are going to scan the created images by using the Aqua-trivy and next by using ansible playbooks we are going to deploy application in k8s.
Following steps in detail:
1.	Create an ubuntu (22.04) t2. Large  instance.
2.	Install Jenkins, docker, ansible and trivy.
a.	To install Jenkins
b.	Install docker
c.	Install trivy
3.	Install plugins like JDK, SonarQube scanner, maven, OWSAP Dependency check
a.	Install plugin
b.	Configure Java and maven in global tool configuration
c.	Create a job
4.	Configure sonar server in manage Jenkins.
5.	Install OWSAP dependency check plugins.
6.	Docker plugin and credential setup
7.	Adding ansible repository in ubuntu
•	Install ansible on ubuntu 22.04 LTS
•	Create an inventory file in ansible.
8.	Kubernetes setup
•	Kubectl on Jenkins to be installed.
Part 1 ----------------- master node ---------------------
           ------------------ worker node ---------------------
Part 2------------------ Both master and node -----------------
Part 3 ------------------master ----------------------------
           ------------------ worker node --------------------
9.	Master slave setup for ansible and Kubernetes.
•	Test ansible master slave connection.
	Here, we will be deploying a pet shop java based application. This is an everyday use case scenario used by several organizations, we will be using Jenkins as a cicd tool and deploying our application on a Docker container and Kubernetes cluster.
	We will be deploying our application in two ways, one using docker container and the other using k8s cluster. 
	Project repo: https://github.com/S-o-n-y/Jpetstore.git
•	Here, in this project I am changing the Jenkins default port number as 8090 why because the java-based application we deploy in tomcat right, so this tomcat webserver runs in port 8080 that’s the reason I am going to change Jenkins default port number.
•	First launch instance with the name main master why because I am not going to create and launch another instance for SonarQube so, that’s the reason I am going to create the SonarQube container from puling of image from docker or with the help of docker I am going to create the SonarQube container.
•	Next select the instance AMI here I am going to use ubuntu OS with the latest version ssd volume and go to instance type an select the t2.large why because I am going to deal with Jenkins and SonarQube in this server or Ec2 instance so each Jenkins and SonarQube both needs the t2. Medium, so, that’s why I am choosing t2.large.
•	Next select the keypair, vpc and subnets as default one or configured one based upon your choice and at security groups select the all traffic why because we have to deal with the more ports that’s why I choose all traffic.
•	Next go to the root volumes and increase the volume size up to 30GB, the reason is we are using the Aqua-Trivy tool for scanning the docker images and related plugins.
•	 Next go to advanced features and go to the user data and add the commands which are going to instal and setup the Jenkins and click on launch instance. 
	Create an ubuntu (22.04) t2.large instance.
	Launch an AWS t2 large instance use the image as ubuntu. You can create a new key pair or use an existing one. Enable HTTPS and HTTP settings in the security group and open all ports
 
2a: To install Jenkins:
	connect to your console, and enter these commands to install Jenkins.
	Here i installed Jenkins through .sh file by using commands 
•	Sudo vi jenkins 
 
	From the below fig can see jenkins installation commands.
•	#!/bin/bash
Sudo apt update -y
Sudo apt-get install openjdk-17-jdk
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y 
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systecmtcl status jenkins
 
	After save and exit from vi shell and give file permissions as 
•	Sudo chmod +x  jenkins.sh
	And execute the .sh file by using the command as 
•	./jenkins.sh
 
	Once Jenkins is installed, you will need to go to you AWS Ec2 security group and open inbound port 8080 and 8090, 9000 for sonar, since Jenkins work on port 8080.
	But for this application case, we are running Jenkins on another port so change the post as 8090 using below commands:
•	Sudo systemctl stop jenkins
•	Sudo systemctl status jenkins
•	Cd /etc/default
•	Sudo vi Jenkins   # change port HTTP_PORT=8090 and save & exit
•	Cd /lib/system/system
•	Sudo vi jenkins.service # change environments= “JENKINS_PORT=8090” save and exit
•	Sudo systemctl deamon-reload
•	Sudo systemctl restart Jenkins
•	Sudo systemctl status jenkins
 
 

 
	Successfully hosted Jenkins with port 8090.
 
2b: Install Docker:
	Now we are going to install and set up the docker for creating the SonarQube container for the update server with commands as <sudo apt update> and <sudo apt-get install docker.io> next to run the docker commands without any admin or sudo permissions we have to add our ubuntu user to the docker group with the commands as <sudo usermod -aG docker (group name) ubuntu(username)> and docker group to take or receive the effect what we add for that use the command as <newgrp docker> and now you have the access without sudo permissions to run the docker.
	Next to push the docker images into docker hub you have to provide full permissions for all users to the /var/run/docker.sock with the command as <sudo chmod 777 /var/run/docker.sock>
	Otherwise you can directly provide 666 permission to the /var/run/docker.sock file it can give the access without sudo permissions to run the docker and to push the docker images into docker hub. 
•	Sudo apt-get update 
•	Sudo apt-get install docker.io -y
•	Sudo usermod -aG docker $username (ubuntu)
•	newgrp docker
•	sudo chmod 777 /var/run/docker.sock
 
 
	After the docker installation we create a SonarQube container (remember added 9000 port in security group)
•	docker run -dt –name sonar -p 9000:9000 sonarqube:lts-community
  
	successfully sonarqube is hosted with port 9000.
	Provide login credentilas username as admin and password as admin.
	After login we can change password. Here I changed admin as sony.
 
	Below fig we can see the official page of SonarQube.
 
	Successfully pushed the SonarQube image into dockerhub.
 
2c: Install Trivy:
	Now, I have to install Aqua-Trivy for docker image scanning for that I am going to create a file or bash file with <sudo vi trivy.sh> and inside this file I am going to write the commands which are going to be install my trivy those commands are:
•	sudo apt-get install wget apt-transport-https gnupg lsb-release
•	wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
•	echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
•	sudo apt-get update
•	sudo apt-get install trivy
	After save the file we have to provide executable permissions for that all users with the command as <sudo chmod +x trivy.sh> and run the file with command as <sudo ./trivy.sh>.
 
 
3a: Install plugin: 
	Go to manage Jenkins  plugins  Available plugins.
	Now, I have to install some plugins which are going to help to deploy pet store in java-based application we have to install java JDK, SonarQube scanner, maven and OWASP dependency check plugins.
	Firstly, I am going to install java JDK plugin for that search in Jenkins plugin with the name Eclipse Termurin Installer (install without restart) and another plugin for SonarQube Scanner for code analysis or checking the code quality for that search in Jenkins plugins with the name SonarQube scanner (install without restart).
 
 
3b: Configure java and maven in global tool configuration:
	Go to manage Jenkins  tools  Install JDK (17) and maven  (3.6.0)  click on apply and save.
	After installing we have to integrate this plugins with Jenkins for that go to global tools options under this option go to the JDK installations. Click on add JDK and give configuration name as JDK17 and select adoptium.net and select the version as JDK- 17.0.8.1+1 and next go to the maven installations and give the configuration name as maven 3 and select version 3.6 and apply and save.
 
 
3c: Create a job:
	Label it as petshop, click on pipeline and ok.
	Now, I am going to create a job for clone the code and compile the code and test the code with maven.
	By using below groovy (or) declarative script create a job in pipeline mode.
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs() 
            }
        }
       stage('checkout scm') {
            steps {
                git 'https://github.com/S-o-n-y/Jpetstore.git'
            }
        }
        stage('maven compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('maven test') {
            steps {
                sh 'mvn test'
            }
        }
   }
}
 
 
  
4.	Configure sonar server in manage jenkins:
	Grab the public ip address of your Ec2 instance SonarQube works on port 9000, so <public ip: 9000>
	Go to sonarQube server, click on Administration  security  users  click on tokerns and update token  give it a name  and click on generate token.
	Now, I am going to configure sonar tool or service with Jenkins we need to token for that, go to the SonarQube dashboard and go to administration and go to security and select users ad go to update token.
 
 
 
	Go to Jenkins dashboard  manage Jenkins  credentials  add secret text.
	In below fig under secret enter or paste the generated token of SonarQube.
 

 
 
	Till now we done only SonarQube configuration only now we have to integrate the SonarQube with the Jenkins for that we have to add details of SonarQube and its version. 
ToolsSonarQube   latest version 5.0.01.3006.
 
	Now, I am going to create the SonarQube quality gate for that go that go to SonarQube official page and click on the configuration and under this select Webhooks and click on create and add a name as Jenkins and under URL field add the Jekins officially running page address along with add SonarQube-webhook/
•	https://Jenkins public ip :8090/sonarqube-webhook/ 
 
 
	Manage Jenkins  system  in SonarQube add (name – sonar server, sonar page ip address:9000, add authentication token).
	Let’s go to pipeline and add SonarQube stage in our pipeline script.
environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=petshop \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=petshop '''
                }
            }
        } 
        stage('quality gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: true, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('TRIVY File scan') {
            steps {
                sh "trivy fs . > trivy-fs_report.txt"
            }
        }
 
 
	Here along with SonarQube quality script I added the Aqua-Trivy script also.
 
 
 
5.	Install OWSAP Dependency check plugins:
	Go to dashboard manage Jenkins  plugins  OWSAP dependency check. Click on it and install it without restart.
 
	After installing we have to integrate this plugins with Jenkins for that go to the global tools options under this option go to the Dependency-Check installations click on add that option and give configured name as DP-Check and select the version before that select the github.com option and add the version as dependency-check 6.5.1.
 
	Let’s go to pipeline and add Dependency check stage in our pipeline script.
        stage('Build war file') {
            steps {
                sh 'mvn clean install -DskipTests=true'
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
 
 
 
6.	Docker plugin and credential setup:
	We need to install Docker tool in our system, go to dashboard  manage plugins  available plugins  search for Docker and install below plugins.
•	Docker
•	Docker commons
•	Docker pipeline
•	Docker ApI
•	Docker build step and click on install without restart
	Docker plugin and credential setup and we need to install the docker tool in our system.
 
 
	After installing we have to integrate this plugins with Jenkins for that go to the global tools options under this option go to the Docker click on add that option and give configured name as docker and select the version before that select the docker.com option and add the version a latest.
 
	Create a personal access token from the docker hub which is used for Ansible playbook.
	Add docker hub username and password under global credentials. Here if you don’t want to add your docker hub account credentials then create a token and use that token as login password of your docker hub account for that go to docker hub account and click on your profile and go to account settings go to security  click new access token and add description of the token and click on create .
 
 
 
 
 
 
7.	Adding ansible repository in ubuntu.
	Now we are going to run the below commands on the Ansible server.
	Update the system packages.
	Adding ansible repository in ubuntu and now we are going to run the below commands on the ansible server for that update your system packages.
	First install required packages to install ansible.
•	Sudo apt install software-properties-common
 

	Add the ansible repository via PPA.
•	Sudo add-apt-repository –yes –update ppa:ansible/ansible
 
	Install ansible in ubuntu
•	Sudo apt install ansible -y
•	Cd /etc/ansible
•	ls
•	Sudo apt install ansible-core -y
 
 
	To add inventory, you can create a new directory or add a default ansible hosts file and now go to the host file inside the ansible server and paste the public ip of the Jenkins.
•	Sudo vi hosts
 
	Install the ansible plugin i.e., Ansible.
	Give the command as which ansible (it shows from which path ansible working) in your Jenkins machine to find the path of your ansible which is used in the tool section of Jenkins.
	Dashboard  manage Jenkins  Tools  path (/usr/bin)
	Add credentials to invoke ansible with Jenkins here we are creating credentials for running ansible playbooks.
	Dashboard  manage Jenkins  credentials  system  global credentials  select SSH username with private key.
	In private key section. Select enter directly add .pem in key.
	Now write an ansible playbook to create a docker image, tag it and push it to the docker hub, and finally, we will deploy it on a container using ansible.
	Just sample code.
 
 
 
 
 
 
 
	Add this stage to the pipeline to build and push it to the docker hub, and run the container.
        stage('AnsiblePlaybook Docker') {
            steps {
                dir ('Ansible') {
                    script {
                        ansiblePlaybook credentialsId: 'ssh', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'docker.yaml'
                    }
                }
            }
        }
        stage('TRIVY') {
            steps {
                sh "trivy image sonydonipati/petstore:latest>trivy.txt"
            }
        }
	In the below fig you have to change the docker stage script at the point playbook: ‘docker-playbook.yaml’ and replace with ‘docker.yml’,
	Here, after ansible related script you have to add the trivy script as:
 
 
 
 
 

 
8.	Kubernetes setup:
•	Connect your machine
•	Take two ubuntu 20.04 instances one for k8s master and other one for worker.
•	Install kubectl on Jenkins machine also:
•	kubectl on Jenkins to be installed (t2 medium).
•	Sudo apt update
•	Sudo apt install curl
•	curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"   
•	sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
•	kubectl version --client
 
Part 2: Both master and worker node:
•	copy
•	sudo apt-get update
•	sudo apt-get install -y docker.io
•	sudo usermod -aG docker ubuntu
•	newgrp docker
•	sudo chmod 777 /var/run/docker.sock
•	sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
•	sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
•	deb https://apt.kubernetes.io/kubernetes-xenial main
•	EOF
•	Sudo apt-get update
•	Sudo apt-get install -y kubelet kubeadm kubectl
•	Sudo snap install kube-apiserver
 
 
 
 
Part 3: master:
	The below commands help to create bridge $ network between the k8 master and k8 nodes.
	Copy
	Sudo kubeadm init  --pod-network -cidr=10.244.0.0/16. # in case your in root exit from it and run below commands.
	mkdir -p $HOME /.kube
	sudo cp -I /etc/Kubernetes/admin .conf $HOME/ .kube/config
	sudo chown $(id -u):$(id -g) $HOME /.kube/config
	kubectl apply -f httpds://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml.
 
 
 
Part 2: Workernode:
	copy
	sudo apt-get update
	sudo apt-get install -y docker.io
	sudo usermod -aG docker ubuntu
	newgrp docker
	sudo chmod 777 /var/run/docker.sock
	sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
	sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
	deb https://apt.kubernetes.io/kubernetes-xenial main
	EOF
	Sudo apt-get update
	Sudo apt-get install -y kubelet kubeadm kubectl
	Sudo snap install kube-apiserver
 
 
 
	Copy
	Sudo kubeadm join <master-node ip>:<master-node-port> --token <token> --discovery-token-ca-cert-hash <hash>
	Here, I am installing kubectl in Jenkins running EC2 instance why because without installing the kubectl we can’t  run the k8s from Jenkins, so, that’s the reason I am installing kubectl in jenkins server by using .sh file with the help of some commands as:
 
Master node:  check the nodes in master i.e., kubectl get nodes.
 

	To make connection between master to ansible-Jenkins server then copy the config file to jenkins master or the local file manager and save it, for that go to k8-master server and <cd.kube> and <cat config>.
	Copy it and save the secret file in the documents or another folder save it as a secret file.txt and create a new text file for the config file as a secret file.txt, store the copied above complete config details and add it in the credentials section.
	Go to dashboard manage Jenkinscredentials system  global credentials. Select the secret file option.
 
 
	Install Kubernetes plugins
•	Kubernetes client API
•	Kubernetes credentials
•	Kubernetes 
•	Kubernetes CLI
•	Kubernetes credential providers
•	Kubernetes pipeline: DevOps steps
 
 
9.	Master- slave set up for ansible and Kubernetes:
	To communicate with the Kubernetes clients, we have to generate an SSH key on the ansible master node and exchange it with k8s master system.
	Here I am going to create a public key and private in the ansible-Jenkins server or instance with the command as <ssh-keygen> and after getting a keys among them I am going to open and copy the public key <id_rsa.pub> and paste that into the k8-master server or instance <cd/ .ssh/authorized-keys> save and exit.
	Next go to instance and copy k8-master server public ip and come back to the ansible Jenkins server and try make a connection  from the ansible to the k8-master server with the command as <ssh ubuntu@ public ip of k8-master server>.
	Note: now, insert or paste the copied public key into the new line make sure don’t delete any existing keys from the authorized keys file then save and exit.
	By adding a public key from the master to the k8s machine we have now configured keyless access. To verify you can try to access the k8s master and use the command as mentioned in the below format.
•	Ssh ubuntu@public ip
	Verifying the above SSH connection from the master to the Kubernetes we have configured our prerequisites.
	Now go to the host file inside the Ansible server and paste the public ip of the k8 master.
•	Cd /etc/ansible
•	Sudo vi hosts 
 
 
 
 
Test Ansible master slave connection: Ansible -m ping k8s (to check ansible-slave connection).
 
	Add this stage to the pipeline to build the kube.yaml.
stage('k8s using ansible') {
            steps {
                dir ('Ansible') {
                    script {
                        ansiblePlaybook credentialsId: 'ssh', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/', playbook: 'kube.yaml'
                    }
                }
            }
        }

 
 
 
 
 








