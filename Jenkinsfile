pipeline{
    agent any
    tools {
      maven 'Maven3.6'
    }
    environment {
      DOCKER_TAG = getVersion()
         // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running. 'nexus-3' is defined in the docker-compose file
        NEXUS_URL = "192.168.205.10:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "myfirstprojectRepo"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "nexus3"
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

                    def mavenPom = readMavenPom file: 'pom.xml'
                 // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
			echo "artifact exist :  ${artifactExists}"
                    nexusArtifactUploader artifacts: [
                        [
                            artifactId: pom.artifactId, 
                            classifier: '', 
                            file: artifactPath, 
                            type: 'war'
                        ]
                    ], 
                    credentialsId: NEXUS_CREDENTIAL_ID, 
                    groupId: pom.groupId, 
                    nexusUrl: NEXUS_URL, 
                    nexusVersion: NEXUS_VERSION, 
                    protocol: NEXUS_PROTOCOL, 
                    repository: NEXUS_REPOSITORY, 
                    version: pom.version
                    }
            }
			
		}
        
        
        stage('Docker Build'){
            steps{
                sh "docker build . -t kd31967/dockeransible:${DOCKER_TAG} "
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
