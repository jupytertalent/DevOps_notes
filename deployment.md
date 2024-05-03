```bash
pipeline {
    agent <agent name>

        stage('basic requirment') {
            steps {
               
                sh "sudo apt install openjdk-17-jdk -y "
                
                sh "sudo apt install maven -y "
               
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
       
        post('clean up docker') {
            always {
               
                sh "docker images | xargs docker rmi -f "
               
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

https://medium.com/@nanditasahu031/jenkins-pipeline-with-maven-sonarqube-and-talisman-fa9118910b98      
