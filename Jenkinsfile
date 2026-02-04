pipeline {
    agent any
    
    tools {
        maven 'maven-3.9'
        jdk 'openjdk-17'
    }

    environment {
        DOCKER_IMAGE = "hklee2748/petclinic"
        DOCKER_HUB_ID = "docker-hub-key"
        K8S_CONFIG_ID = "k8s-config"
    }
    
    stages {
        stage('1. Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], 
                    userRemoteConfigs: [[credentialsId: 'github-ssh-key', url: 'git@github.com:kwony93/k8s-test.git']])
            }
        }

        stage('2. Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
            post {
                success { echo 'Maven Build Success' }
                failure { echo 'Maven Build Failed' }
            }
        }

        stage('3. Docker Push') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKER_HUB_ID}") {
                        def appImage = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                        appImage.push()
                        appImage.push("latest")
                    }
                }
            }
        }

        stage('4. K8s Deploy') {
            steps {
                withKubeConfig([credentialsId: "${K8S_CONFIG_ID}"]) {
                    // 환경변수 처리를 더 안전하게 하기 위해 더블 쿼트 확인
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${BUILD_NUMBER}|g' k8s/deployment.yaml"
                    sh 'kubectl apply -f k8s/deployment.yaml'
                    sh "kubectl rollout status deployment petclinic"
                }
            }
        }

        stage('5. Cleanup (Local Only)') {
            steps {
                echo 'Cleaning up local images...'
                // 빌드 시 사용한 특정 태그 이미지 삭제
                sh "docker rmi -f ${DOCKER_IMAGE}:${BUILD_NUMBER} || true"
                sh "docker rmi -f ${DOCKER_IMAGE}:latest || true"
                sh "docker image prune -f"
            }
        }
    }

    post {
        success { echo "Successfully deployed build #${BUILD_NUMBER}" }
        failure { echo "Deployment failed. Check the logs." }
    }
}
