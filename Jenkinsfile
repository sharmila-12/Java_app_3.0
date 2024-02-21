@Library('my-shared-library') _

pipeline{

    agent any

    parameters{

        choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        string(name: 'ImageName', description: "name of the docker build", defaultValue: 'javapp')
        string(name: 'ImageTag', description: "tag of the docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "name of the Application", defaultValue: '7578957ca1f9')
    }

    stages{

        stage('Git Checkout'){
                    when { expression {  params.action == 'create' } }
            steps{
            gitCheckout(
                branch: "main",
                url: "https://github.com/sharmila-12/Java_app_3.0.git"
            )
            }
        }
         stage('Unit Test maven'){

         when { expression {  params.action == 'create' } }

            steps{
               script{

                   mvnTest()
               }
            }
        }
         stage('Integration Test maven'){
         when { expression {  params.action == 'create' } }
            steps{
               script{

                   mvnIntegrationTest()
               }
            }
        }
        stage('Static code analysis: Sonarqube'){
         when { expression {  params.action == 'create' } }
            steps{
               script{

                   def SonarQubecredentialsId = 'sonarqube-api'
                   statiCodeAnalysis(SonarQubecredentialsId)
               }
            }
       }
       stage('Quality Gate Status Check : Sonarqube'){
         when { expression {  params.action == 'create' } }
            steps{
               script{

                   def SonarQubecredentialsId = 'sonarqube-api'
                   QualityGateStatus(SonarQubecredentialsId)
               }
            }
       }
        stage('Maven Build : maven'){
         when { expression {  params.action == 'create' } }
            steps{
               script{

                   mvnBuild()
               }
            }
        }
       // stage ('Pushing Jar to Jfrog : python'){
      //    when { expression {  params.action == 'create' } }
        //  steps{
          //  script{
            //    jfrogPush()
       //         }
     //       }
     //   }
        stage('Docker Image Build'){
         when { expression {  params.action == 'create' } }
            steps{
               script{

                   dockerBuild("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }
         stage('Docker Image Scan: trivy '){
         when { expression {  params.action == 'create' } }
            steps{
               script{

                   dockerImageScan("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }
        stage ('Pushing Jfrog File'){
          when { expression {  params.action == 'create' } }
          steps{
            script{
                 sh 'curl -X PUT -u admin:password -T  /var/lib/jenkins/workspace/java-3.0/target/kubernetes-configmap-reload-0.0.1-SNAPSHOT.jar "http://35.174.204.213:8082/artifactory/example-repo-local/kubernetes-configmap-reload-0.0.1-SNAPSHOT.jar"'
                }
            }
        }
        stage('Docker Image Push : DockerHub '){
         when { expression {  params.action == 'create' } }
            steps{
               script{

                   dockerImagePush("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }   
        stage('Docker Image Cleanup : DockerHub '){
         when { expression {  params.action == 'create' } }
            steps{
               script{

                   dockerImageCleanup("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }
    }
}
