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
                // 아까 만든 SSH 자격 증명(github-ssh-key) 사용
                checkout scmGit(branches: [[name: '*/main']], 
                    extensions: [], 
                    userRemoteConfigs: [[credentialsId: 'github-ssh-key', 
                    url: 'git@github.com:<네-아이디>/k8s-test.git']])
            }
        }

        stage('2. 빌드 (Maven Build)') {
            steps {
                // 펫클리닉 JAR 파일 만들기
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('3. 이미지 굽기 (Docker Build)') {
            steps {
                // 나중에 도커 허브나 레지스트리에 올릴 이미지 만들기
                sh 'docker build -t hklee2748/petclinic:${BUILD_NUMBER} .'
            }
        }

        stage('4. 배포 (K8s Deploy)') {
            steps {
                // 쿠버네티스에 배포하기 (나중에 deployment.yaml 필요함)
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
}
