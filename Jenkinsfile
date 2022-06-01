pipeline{
    agent any
    tools{
        jdk "jdk8"
    }
    environment{
        registryCredential = 'dockerhub'
        registry = "ctacristos/vprofile_repo"

    }
    stages {
        stage('fetch code'){
            steps{
                git branch: 'main', url: 'https://github.com/CTACRISTOS/cicd_kube.git'
            }

        }
        stage('Test'){
            steps{
                sh 'mvn test'
            }
        }
        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage('CODE ANALYSIS with SONARQUBE') {
            tools{
                jdk "jdk11"
            }

            environment {
                scannerHome = tool 'sonar4.7'
            }

            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('build multistage image') {
            steps{
                script{
                    dockerImage = docker.build( registry + ":$BUILD_ID", "./dockermultistage/")

                }
            }
        }
        stage('upload image to nexus'){
            steps{
                script {
                    docker.withRegistry('', registryCredential){
                        dockerImage.push("$BUILD_ID")
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('remove local image') {
            steps{
                sh "docker rmi $registry:$BUILD_ID"
            }
        }
        stage('K8s Deploy'){
            steps{
                sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:{BUILD_ID} --namespace pipehelm"
            }
        }
    }
}