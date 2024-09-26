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
                   withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh "aws s3 cp s3://javabkt/LoginWebApp.war ."
                    sh "sudo docker image prune -a -f"
                    sh "ls -lrt"
                    sh "pwd"
                    sh "sudo docker build -t gyana545/mysql:v1 -f dockermysql ."
                    sh "sudo docker build -t gyana545/tomcat:v3 -f dockertomcat ."
                    sh 'sudo docker login -u "gyana545" --password gyana@1996 docker.io'
                    sh "sudo docker push gyana545/mysql:v1"
                    sh "sudo docker push gyana545/tomcat:v3"
                   }
                }
            }
        }
        stage(deploy through ansible){
            agent{
                label 'ansible'
            }
            steps{
                script{
                   sh "cd Ansible/ && ansible-playbook Java_Deploy.yml"
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

//         stage('Deployments') {
//             steps {
//                 script {
//                     // deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://43.204.98.141:8080/')], 
//                     //        contextPath: null, 
//                     //        war: '**/*.war'
//                     sh "ls -l && pwd"
//                     withCredentials([sshUserPrivateKey(credentialsId: 'docker_server', usernameVariable: 'USERNAME', keyFileVariable: 'KEYFILE')]) {
//                         script {
//                             def remote = [:]
//                             remote.name = 'docker_server'
//                             remote.host = '13.233.103.234'
//                             remote.allowAnyHosts = true
//                             remote.user = USERNAME
//                             remote.identityFile = KEYFILE
//                             sshCommand remote: remote, command: 'ls -l && pwd'
//                             sshPut remote: remote, from: 'target/LoginWebApp.war', into: './java'
//                             sshPut remote: remote, from: 'dump/devopsclass.sql', into: './java'
//                             sshPut remote: remote, from: 'Dockerfile-tomcat', into: './java'
                            
//                             // Uncomment below lines when ready to use Docker
//                             sshCommand remote: remote, command: 'sudo docker build -t tomcat-lipu ./java'
//                             sshCommand remote: remote, command: 'sudo docker run -d -p 8080:8080 --name tomcat tomcat-lipu'
//                             // sshCommand remote: remote, command: 'sudo docker rmi tomcat-8.0'
//                             // sshRemove remote: remote, path: 'DockerJavaApp/LoginWebApp.war'
//                             // sshRemove remote: remote, path: 'DockerJavaApp/tomcat'
//                         }
//                     }
//                 }
//             }
//         }
    }
}