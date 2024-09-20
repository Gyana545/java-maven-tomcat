pipeline {
    agent any

    tools {
        maven 'maven'
    }

    stages {
        stage('SCM Checkout') {
            steps {
                sh 'whoami'
                git branch: 'master', url: 'https://github.com/Gyana545/LoginWebApp.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
            post {
                success {
                    echo 'Archiving the artifacts'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Deployments') {
            steps {
                script {
                    // deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://43.204.98.141:8080/')], 
                    //        contextPath: null, 
                    //        war: '**/*.war'
                    withCredentials([sshUserPrivateKey(credentialsId: 'docker_server', usernameVariable: 'USERNAME', keyFileVariable: 'KEYFILE')]) {
                        script {
                            def remote = [:]
                            remote.name = 'docker_server'
                            remote.host = '13.233.103.234'
                            remote.allowAnyHosts = true
                            remote.user = USERNAME
                            remote.identityFile = KEYFILE
                            sshCommand remote: remote, command: 'ls -l && pwd'
                            sshPut remote: remote, from: 'target/LoginWebApp.war', into: './java', sudo: true
                            // sshPut remote: remote, from: 'dump/devopsclass.sql', into: './java'
                            // sshPut remote: remote, from: 'tomcat', into: './java'
                            
                            // Uncomment below lines when ready to use Docker
                            // sshCommand remote: remote, command: 'sudo docker build -t tomcat-8.0 -f tomcat ./DockerJavaApp'
                            // sshCommand remote: remote, command: 'sudo docker run -d -p 8080:8080 --name tomcat tomcat-8.0'
                            // sshCommand remote: remote, command: 'sudo docker rmi tomcat-8.0'
                            // sshRemove remote: remote, path: 'DockerJavaApp/LoginWebApp.war'
                            // sshRemove remote: remote, path: 'DockerJavaApp/tomcat'
                        }
                    }
                }
            }
        }
    }
}
