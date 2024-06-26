pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "sharonba/vprofileapp"
        registryCredential = 'dockerhub'
    }

    stages{
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }


        stage('Build Docker App Image'){
            steps {
                script {
                    dockerImage = docker.build(registry + ":V$BUILD_NUMBER")
                }
            }

        }

        stage("Upload image"){
           steps{
              script{
                    echo 'Starting to push'
                    docker.withRegistry('','dockerhub'){
                       
                        dockerImage.push("V$BUILD_NUMBER")
                        dockerImage.push("latest")

                    }
                
                }
            }
        }

        stage("Remove unused docker images"){
            steps{
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }

        stage("Kubernetes Deploy"){
            agent {label 'KOPS'}
                steps{
                    sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER}"
                }
        }


    }


}
