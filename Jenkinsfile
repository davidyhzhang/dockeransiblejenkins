pipeline{
    agent any
    tools {
      maven 'maven'
    }
    environment {
      servers = 'webservers'
      DOCKER_TAG = getVersion()
      // DOCKER_TAG = '0.01'
      USER = "davidyhzhang"
    }
    stages{
        stage('SCM'){
            steps{
                git credentialsId: 'github', 
                    url: 'https://github.com/davidyhzhang/dockeransiblejenkins'
            }
        }
        
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build'){
            steps{
                sh "docker build . -t ${USER}/demo-app:${DOCKER_TAG} "
            }
        }
        
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u ${USER} -p ${dockerHubPwd}"
                }
                
                sh "docker push ${USER}/demo-app:${DOCKER_TAG} "
            }
        }
        
        stage('Docker Deploy'){
            steps{
                /**ansiblePlaybook credentialsId: 'dev-server', 
                    disableHostKeyChecking: true, 
                    extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', 
                    inventory: 'dev.inv', playbook: 'deploy-docker.yml'
                */
                ansiblePlaybook disableHostKeyChecking: true, 
                    extras: "-e DOCKER_TAG=${DOCKER_TAG} -e servers=${servers} -e USER=${USER}", 
                    installation: 'ansible', inventory: 'dev.inv', 
                    playbook: 'deploy-docker.yml'            }
        } 
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, 
        script: 'git rev-parse --short HEAD'
    return commitHash
}
