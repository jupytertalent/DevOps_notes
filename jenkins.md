Jenkins is an open-source automation server that enables developers around the world to reliably build, test, and deploy their software. It is a highly flexible and versatile tool that can handle a wide variety of tasks, making it an invaluable asset in many parts of the software development process. It provides hundreds of plugins to support building, deploying and automating any project. With Jenkins, organizations can accelerate the software development process through automation. Furthermore, Jenkins integrates with practically every tool in the continuous integration and continuous delivery toolchain.

```bash
sudo apt install openjdk-17-jdk -y 

	curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
	
	echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
	
sudo apt update

sudo apt install jenkins -y

sudo systemctl status jenkins

sudo systemctl enable --now jenkins
```

change the rule inbound rule open a 8080 and 50000 for exposing and Listening

---

**In Docker**

```bash
docker pull jenkins/jenkins
```

```bash
docker run -p 8080:8080 -p 50000:50000 -v /opt:/var/jenkins_home jenkins/jenkins
```

---

Basic idea about is it will run in shell only if you want installation or any automative things means we can use this other wise cli application inside running is not possible 

declarative and scripted pipeline 

both looks same but security wise scripted more usefull

# pipeline

```
pipeline {
agent any
		stages {
		    stage('Checkout') {
	        steps {
            // Checkout source code from your version control system
            git '<https://github.com/your/repository.git>'
        }
    }
}
    stage('Build') {
        steps {
            // Build your application (replace with your build commands)
            sh 'mvn clean package' 
        }
    }

    stage('Test') {
        steps {
            // Run your tests (replace with your test commands)
            sh 'mvn test'
        }
    }

    stage('Deploy') {
        steps {
            // Deploy your application to your deployment environment (replace with your deployment commands)
            sh 'ssh -i $ssh-key user@yourserver "cd /path/to/deployment && ./deploy.sh"'
        }
    }
}

```

Queueing of Job → upstream/ downstream

priodic trigger (crontab)

remote trigger (url to trigger the job)

1. Post build to build other project then select a job name 
2. crontab schedules crontab.guru
3. build trigger to trigger a job 

plugin : role based authorization strategy

```
delivery pipeline
build pipeline           # user level build setup visual view
build monitor 
```

 plugin:

sonarscanner

git server

maven integration

amazon ec2

jdk : exlipse temurin installer (if the project is jdk base )

junit plungin

# Git integration

git integration for that you should download a git plung in 

inside of ci/cd select a git and give a git url then give a credentials now its ready 

The jenkins ci/cd file execute in workspace folder 

the location is / var/lib/jenkins 

jenkins port change 

/etc/default/jenkins

# Tomcat and jenkins integration

https://tomcat.apache.org/download-90.cgi    tomcat download link

wget [https://downloads.apache.org/tomcat/tomcat-9/v9.0.87/bin/apache-tomcat-9.0.87.tar.gz](https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.87/src/apache-tomcat-9.0.87-src.tar.gz)

tar -xzvf [apache-tomcat-9.0.87-src.tar.gz](https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.87/src/apache-tomcat-9.0.87-src.tar.gz)

you got a folder navigate to bin folder /bin

[startup.sh](http://startup.sh) to start the service

tomcat default port change 

/apache-tomcat.x.x.x/conf/server.xml

gui selection a war file location host details 

# Role Auth

manage role to set a user permission 

first things is make a users 

then set a permission for that particular user 

# Master and Slave Integration

Master installation usual so this is slave installation 

sudo apt install openjdk-11-jdk -y

https://medium.com/@dksoni4530/jenkins-master-slave-architecture-setup-f0486fba8039

cloud node or node setup to lauch a ssh setup

execution and remote path important 

then norml cicd things 

# Gitlab Ci/CD

```yaml
deploy:
    stage : deploy
    before_script:
        - chmod 600 $ssh_key
    script:
    - ssh -o StrictHostKeyChecking=no -i $ssh_key ubuntu@3.27.4.162 "touch test_case.txt"

```

IF you got like this set a listener port fix 50000 

Mar 31, 2024 5:37:46 AM hudson.remoting.Launcher$CuiListener status
INFO: Locating server among [http://3.107.8.52:8080/]
Mar 31, 2024 5:37:46 AM hudson.remoting.Launcher$CuiListener status
INFO: Could not locate server among [http://3.107.8.52:8080/]; waiting 10 seconds before retry
java.io.IOException: http://3.107.8.52:8080/tcpSlaveAgentListener/ is invalid: 404 Not Found
at org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver.resolve(JnlpAgentEndpointResolver.java:222)
at hudson.remoting.Engine.innerRun(Engine.java:809)
at hudson.remoting.Engine.run(Engine.java:563)

# set a lable its a important

remote the server task

```yaml
pipeline{
		agent any
tools {
		maven 'localMaven'
}
stages{
		stage('Build'){
		steps{
			sh 'mvn clean package'
			sh 'scp Dockerfile [centos@3.17.61.170](mailto:centos@3.17.61.170)'
			sh 'ssh [centos@3.17.61.170](mailto:centos@3.17.61.170) "docker build . -t tomcatwebapp:${env.BUILD_ID}"'
		}
	}
	}
}
```

# full deployment with docker

```yaml
pipeline {
    agent any

        stage('basic requirment') {
            steps {
             
                sh "sudo apt update"
            
                sh "sudo apt install openjdk-17-jdk -y "
                
                sh "sudo apt install maven -y "
                
                sh "sudo apt install docker.io -y "
               
            }
        }
    stages {
        stage('Checkout') {
            steps {
                // clone the repo
               git branch: 'main', url: 'https://github.com/Voilater/Spring_Boot_Demo'
            }
        }

        stage('Build') {
            steps {
                // Build the project using Maven
                sh "/usr/bin/mvn clean package"
            }
        }

        stage('image building') {
            steps {
               
                sh "docker build -t selflearning ."
               
            }
        }
        
        stage('ECR push') {
            steps {
               
                sh "aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 654654487652.dkr.ecr.ap-southeast-2.amazonaws.com"
                
                sh "docker tag selflearning:latest 654654487652.dkr.ecr.ap-southeast-2.amazonaws.com/selflearning:latest"
                
                sh "docker push 654654487652.dkr.ecr.ap-southeast-2.amazonaws.com/selflearning:latest"
               
            }
        }
        
        stage('Sonarqube scan') {
            steps {
               
                sh "mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=spring_boot \
                    -Dsonar.projectName='spring_boot' \
                    -Dsonar.host.url=http://54.252.30.195:9000 \
                    -Dsonar.token=sqp_3b838206451dc00338592b5a301ffa879a1b677c"
               
            }
        }
    }
        stage('clean up docker') {
            steps {
               
                sh "docker images | xargs docker rmi -f "
                
                sh "docker ps -a | xargs docker stop "
                
                sh "docker ps -a | xargs docker rm "
               
            }
        }
        stage('deployment ') {
            steps {
                // Deploy the code 
                sh "docker pull 654654487652.dkr.ecr.ap-southeast-2.amazonaws.com/selflearning:latest"
                
                sh "docker run -itd -p 80:8082 --name myapp 654654487652.dkr.ecr.ap-southeast-2.amazonaws.com/selflearning:latest"
            }
        }
        
    post {
        success {
            // Actions to perform after successful pipeline execution
            echo 'Pipeline successfully completed!'
            // You may want to send notifications or trigger other jobs here
        }

        failure {
            // Actions to perform if the pipeline fails
            echo 'Pipeline failed!'
            // You may want to send notifications or perform cleanup here
        }
    }
}

```