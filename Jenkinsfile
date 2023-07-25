pipeline {
  agent any
  tools {
  
  maven 'Maven'
   
  }
    stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/adegokeobafemi/jan-devops.git']]])
        }
      }
	  
	  stage ('Build')  {
	      steps {
            dir('webapp'){
            sh "pwd"
            sh "ls -lah"
            sh "mvn package"
          }
        }
         
      }
   
     stage ('SonarQube Analysis') {
        steps {
              withSonarQubeEnv('sonar') {
                
				dir('webapp'){
                 sh 'mvn -U clean install sonar:sonar'
                }
				
              }
            }
      }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog",
                    url: "http://44.213.18.133:8082/artifactory",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "jan-devops-libs-release-local",
                    snapshotRepo: "jan-devops-libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog",
                    releaseRepo: "jan-devops-libs-release",
                    snapshotRepo: "jan-devops-libs-snapshot"
                )
            }
    }

    stage ('Deploy Artifacts') {
            steps {
                rtMavenRun (
                    tool: "Maven", // Tool name from Jenkins configuration
                    pom: 'webapp/pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
         }
    }

    stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "jfrog"
             )
        }
    }

    stage('Copy Dockerfile & Playbook to Ansible Server') {
            steps {
	    sshagent(['ssh_agent']) {
                        sh "chmod 400  Itern-KP.pem"
                        sh "ls -lah"
                	sh "scp -i Itern-KP.pem -o StrictHostKeyChecking=no Dockerfile ubuntu@52.202.178.144:/home/ubuntu"
                        sh "scp -i Itern-KP.pem -o StrictHostKeyChecking=no dockerfile ubuntu@52.202.178.144:/home/ubuntu"
                        sh "scp -i Itern-KP.pem -o StrictHostKeyChecking=no devops.yaml ubuntu@52.202.178.144:/home/ubuntu"
                    }
                }
        } 
  
 stage('Build Container Image') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "ssh -i Itern-KP.pem -o StrictHostKeyChecking=no ubuntu@52.202.178.144 -C \"ansible-playbook  -vvv -e build_number=${BUILD_NUMBER} devops.yaml\""
                    }
                }
        } 
	    
	    
    stage('Copy Deployment & Service Defination to K8s Master') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "scp -i Itern-KP.pem -o StrictHostKeyChecking=no deploy.yml ubuntu@18.205.98.141:/home/ubuntu"
                        sh "scp -i Itern-KP.pem -o StrictHostKeyChecking=no service.yml ubuntu@18.205.98.141:/home/ubuntu"
                    }
                }
            
        } 

    stage('Waiting for Approvals') {
            
        steps{

				input('Test Completed ? Please provide  Approvals for Prod Release ?')
			  }
            
    }     
    stage('Deploy Artifacts to Production') {
            
            steps {
                  sshagent(['ssh_key']) {
                        sh "ssh -i Itern-KP.pem -o StrictHostKeyChecking=no ubuntu@18.205.98.141 -C \"kubectl set image deployment/itern-deploy iterncon=adegokeobafemi/lab:${BUILD_NUMBER}\""
                       //sh "ssh -i Itern-KP.pem -o StrictHostKeyChecking=no ubuntu@18.205.98.141 -C \"kubectl delete pod itern-deploy\""
                        sh "ssh -i Itern-KP.pem -o StrictHostKeyChecking=no ubuntu@18.205.98.141 -C \"kubectl apply -f deploy.yml\""
                        sh "ssh -i Itern-KP.pem -o StrictHostKeyChecking=no ubuntu@18.205.98.141 -C \"kubectl apply -f service.yml\""
                        
                    }
                }
            
        } 
         
   } 
}
