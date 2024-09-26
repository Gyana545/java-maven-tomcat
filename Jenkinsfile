pipeline {
    agent any

    tools {
        maven 'maven'
    }

    stages {
        stage('SCM Checkout') {
            steps {
                sh "ls -l && pwd"
                sh 'whoami'
                git branch: 'master', url: 'https://github.com/Gyana545/LoginWebApp.git'
            }
        }
    }

        stage('Build') {
            steps {
                sh "ls -l && pwd"
                sh 'whoami'
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'Archiving the artifacts'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        // stage ('upload to s3'){
        //     steps{
        //        withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
        //         sh "aws s3 ls"
        //         sh "aws s3 cp target/*.war s3://mbtechbucket/techdb/" 
        //         } 
        //     }
        // }
         stage('Upload to S3 with s3 plugin') {
            steps {
                script {
                    s3Upload consoleLogLevel: 'INFO', 
                    dontSetBuildResultOnFailure: false, 
                    dontWaitForConcurrentBuildCompletion: false, 
                    entries: [[
                        bucket: 'javabkt', 
                        excludedFile: '', 
                        flatten: false, 
                        gzipFiles: false, 
                        keepForever: false, 
                        managedArtifacts: false,
                        noUploadOnFailure: false, 
                        selectedRegion: 'ap-south-1', 
                        showDirectlyInBrowser: false, 
                        sourceFile: 'target/*.war', 
                        storageClass: 'STANDARD', 
                        uploadFromSlave: false, 
                        useServerSideEncryption: true
                    ]], 
                    pluginFailureResultConstraint: 'FAILURE', 
                    profileName: 's3_gyana', 
                    userMetadata: []
                }
            }
        }
        stage('download from s3 through agent'){
            agent{
                label 'docker_node'
            }
            steps{
                script{
                   withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]){
                    sh """
                    aws s3 cp s3://javabkt/LoginWebApp.war .
                    sudo docker image prune -a -f
                    ls -lrt
                    pwd
                    sudo docker build -t gyana545/mysql:v1 -f dockermysql .
                    sudo docker build -t gyana545/tomcat:v3 -f dockertomcat .
                    sudo docker login -u "gyana545" --password gyana@1996 docker.io
                    sudo docker push gyana545/mysql:v1
                    sudo docker push gyana545/tomcat:v3
                   """
                   }
                }
            }
        stage('deploy through ansible'){
            agent{
                label 'ansible'
            }
            steps{
                script{
                   sh "sudo ansible-playbook java_app_deploy.yml"
                   cleanWs()
                   echo "clean Workspace"
                }
            }
        }
        stage('clean Workspace') {
            steps {
                cleanWs()
                echo "clean ws"
            }
        }
    }
}



