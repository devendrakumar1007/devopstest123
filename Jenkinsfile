pipeline{
    agent any
    tools {
      maven 'Maven3.6'
    }
    environment {
      DOCKER_TAG = getVersion()
    }
    options {
        buildDiscarder logRotator(daysToKeepStr: '5', numToKeepStr: '7')
    }
    stages{
        stage('SCM'){
            steps{
                git credentialsId: 'github', 
                    url: 'https://github.com/devendrakumar1007/devopstest123.git'
            }
        }
        
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
                 archiveArtifacts artifacts: 'target/*.war', onlyIfSuccessful: true
            }
        }
        
             stage('Upload War To Nexus'){
            steps{
                script{

                    //def mavenPom = readMavenPom file: 'pom.xml'
                   // def nexusRepoName = mavenPom.version.endsWith("SNAPSHOT") ? "simpleapp-snapshot" : "simpleapp-release"
                    nexusArtifactUploader artifacts: [
                        [
                            artifactId: 'dockeransible', 
                            classifier: '', 
                            file: "target/dockeransible-1.0-SNAPSHOT.war", 
                            type: 'war'
                        ]
                    ], 
                    credentialsId: 'nexus3', 
                    groupId: 'in.javahome', 
                    nexusUrl: '192.168.205.10:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'myfirstprojectRepo', 
                    version: '1.0-SNAPSHOT'
                    }
            }
			
		}
        
        
        stage('Docker Build'){
            steps{
                sh "docker build . -t kd31967/webapp:${DOCKER_TAG} "
            }
        }
        // docker hub plugin install and set the password in for docker repository in jenkins credential
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u kd31967 -p ${dockerHubPwd}"
                }
                
                sh "docker push kd31967/dockeransible:${DOCKER_TAG} "
            }
        }
        
        stage('Docker Deploy'){
            steps{
             // ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            sh "echo hi "
            
            }
        }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
