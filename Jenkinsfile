pipeline {
    agent any
tools {
  maven 'maven_home'
  dockerTool 'docker'
  
}
environment {
        BASE_VERSION = "v1"
        registry = "sahanabhasme19/mavenbuild"
        DOCKER_CREDENTIALS = credentials('dockpass')
        DOCKER_IMAGE = 'sahanabhasme19/mavenbuild:v1'
        ARM_CLIENT_SECRET = credentials('client_secret')
        ARM_CLIENT_ID = credentials('client_id')
        ARM_SUBSCRIPTION_ID = credentials('sub_id')
        ARM_TENANT_ID = credentials('ten_id')
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

        stage('Code quality check') {
            steps {
                withSonarQubeEnv('sonar') {
			     sh "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar -Dsonar.properties=sonar-project.properties -Dsonar.projectKey=com.java:MavenBuild -Dsonar.projectName=Devops_squad_5"
		}
            }
        }
        
        stage("deploy to tomcat")
        {
		// when {
  //               expression { 
  //                   env.BRANCH_NAME == 'master'
  //               }
		// }
            steps{
                script{
                    deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://13.82.21.94:8080')], contextPath: 'devops', war: 'target/*.war'
                }
            }
        }
        
        stage('Building our image') {
		// when {
		// 	branch 'master'
		// }
            steps{
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        }
        stage('Deploy our image') {
		// when {
		// 	branch 'master'
		// }
            steps{
                script {
                    sh 'docker login -u sahanabhasme19 -p ${DOCKER_CREDENTIALS}'
                    sh 'docker push ${DOCKER_IMAGE}' 
                    // docker.image(DOCKER_IMAGE).push('latest')
                }
            }
        }
        stage('Cleaning up') {
		// when {
		// 	branch 'master'
		// }
            steps{
                sh "docker rmi sahanabhasme19/mavenbuild:v1"
                // 
            }
        }
        stage('Aks Context set') {
		// when {
		// 	branch 'master'
		// }
            steps{
                sh "az aks get-credentials --resource-group aks-k8s-resources --name aks-k8s"
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl delete hpa devopssquad5"
                sh "kubectl autoscale deployment devopssquad5 --cpu-percent=50 --min=1 --max=3"
                sh "kubectl get svc"
                sh "kubectl get nodes"
            }
        }
    }
    

        
}
