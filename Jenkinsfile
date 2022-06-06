def globalDynamicVars=[:]
pipeline{
    agent any
    tools{
        jdk "jdk8"
    }
    environment{
        registryCredential = 'dockerhub'
        registry = "ctacristos/vprofileimg"

    }
    stages {
        stage('fetch code'){
            steps{
                git branch: 'main', url: 'https://github.com/CTACRISTOS/cicd_kube.git'
            }

        }
        stage('Test'){
            steps{
                script{
                    def pom=readMavenPom file: 'pom.xml'

                    globalDynamicVars.appName=pom.artifactId
                    globalDynamicVars.appVersion=pom.version
                    globalDynamicVars.imageTag="${globalDynamicVars.appVersion}-${BUILD_NUMBER}"
                    globalDynamicVars.registry=registry
                }
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
                    dockerImage = docker.build( globalDynamicVars.registry + ":"+globalDynamicVars.imageTag)

                }
            }
        }
        stage('upload image to dockerhub'){
            steps{
                script {
                    docker.withRegistry('', registryCredential){
                        dockerImage.push(globalDynamicVars.imageTag)
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('remove local image') {
            steps{
                sh "docker rmi ${globalDynamicVars.registry}:${globalDynamicVars.imageTag}"
            }
        }

    }
}
podTemplate(containers: [
        containerTemplate(
                name: 'jnlp',
                image: 'jenkins/inbound-agent:latest'
        ),

        containerTemplate(
                name: 'helm',
                image: 'alpine/helm',
                command: 'sleep',
                args: '30d'
        )

]) {

    node(POD_LABEL) {
        stage('helm deploy') {

            container('helm') {
                stage('helm job') {
                    git branch: 'main', url: 'https://github.com/CTACRISTOS/cicd_kube.git'
                    sh '''
                    echo "helm"
                    '''
                    sh '''
                    helm version
                    '''
                    sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${globalDynamicVars.registry}:${globalDynamicVars.imageTag} --namespace clean"
                }
            }
        }



    }
}