pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        jdk 'jdk-17'
        maven 'maven3'
    }
    
    environment {
        DOCKER_CREDENTIALS_ID = "docker-credentials"
        DOCKER_REGISTRY = "bazzoumohammed"
        DOCKER_IMAGE_NAME = "shopping"  
        DOCKER_IMAGE_TAG = "latest"
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/bazzou-mohammed/Spring-Boot-Shopping-Cart-Web-App.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        //sonar scanner using maven plugin
        stage('SonarQube analysis') {
            steps {
                // Execute SonarQube analysis
                script {
                    withSonarQubeEnv('sonar_server') {
                        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                    }
                }
            }
        }
        
        stage('Build APP') {
            steps {
                sh "mvn clean install -DskipTests=true"
            }
        }
        
        stage('Build & Push Docker image') {
            steps {
                script {
                    // Login to Docker Hub
                    withDockerRegistry(credentialsId: DOCKER_CREDENTIALS_ID) {
                      sh '''
                        docker build -t ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} -f docker/Dockerfile .
                        docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                        '''
                    }
                }
            }
        }

        stage('Docker Deploy To Container') {
            steps {
                script {
                    // Login to Docker Hub
                    withDockerRegistry(credentialsId: DOCKER_CREDENTIALS_ID) {
                      sh 'docker run -d --name shopping-cart -p 8070:8070 ${DOCKER_REGISTRY}/${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}' 
                    }
                }
            }
        }


        // stage('Deploy To Tomcat server') {
        //     steps {
        //         script {
        //             // Deploy the WAR file to Tomcat server
        //             deploy adapters: [tomcat9(credentialsId: 'deployer', path: '', url: 'http://localhost:8081/')], contextPath: 'java-spring-app', war: '**/*.jar'
        //         }
        //     }
        // }
    }
}
