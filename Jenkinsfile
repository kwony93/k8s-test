pipeline {
    agent any
    
    // 우리가 설정한 도구들 불러오기
    tools {
        maven 'maven-3.9'
        jdk 'openjdk-17'
    }

    stages {
        stage('1. 준비 (Checkout)') {
            steps {
                // 깃허브 인증
                checkout scmGit(branches: [[name: '*/main']], 
                    extensions: [], 
                    userRemoteConfigs: [[credentialsId: 'github-ssh-key', 
                    url: 'git@github.com:kwony93/k8s-test.git']])
            }
        }

        stage('2. 빌드 (Maven Build)') {
            steps {
                // 펫클리닉 JAR 파일 만들기
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('3. 이미지 굽기 및 푸시') {
            steps {
                // 도커 허브용 이미지 생성
                script {
                    docker.withRegistry('', 'docker-hub-key') {
                        def appImage = docker.build("hklee2748/petclinic:${BUILD_NUMBER}" )
                        appImage.push()
                        appImage.push("latest")
                    }
                }
            }
        }
            // Docker Image Remove
        stage('4. Docker Image Remove') {
          steps {
            echo 'Docker Image Remove'
            sh 'docker rmi -f hklee2748/petclinic:$BUILD_NUMBER'
          }
        }
        
        stage('5. 배포 (K8s Deploy)') {
            steps {
                // 쿠버네티스에 배포하기 (나중에 deployment.yaml 필요함)
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
}
