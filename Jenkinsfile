pipeline {
    agent any
tools {
  maven 'M3'
  
}
    
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
		    echo "Current branch: ${env.BRANCH_NAME}"
            }
        }
        stage('Build Automation') {
		
            steps {
                sh "mvn clean package"
                archive 'target/*.war'
            }
        }
        stage('Unit test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                        }
                }
        }

       
        stage("deploy to tomcat")
        {
		when {
                expression { 
                    env.BRANCH_NAME == 'master'
                }
		}
            steps{
                script{
                    deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://13.82.21.94:8080')], contextPath: 'devops', war: 'target/*.war'
                }
            }
        }
        
       
    }
    

        
}
