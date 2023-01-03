/*
Triggers:
  1. For merge commit via pull request in main branch - Yes - (Triggers job based on configuration in Jenkins job)
  2. For commits in feature/bugfix branches - No - (Skips trigger based on configuration in Jenkikns job)
  3. For Opening Pull Requests - Yes (but skip certain stages) - (Triggers job based on Jenkins job config but skips certain stages such as deployment
  with the help of script in Jenkinsfile)
*/

def checkoutDetails;
def isPullRequest = true;
pipeline {
	agent {
  	  kubernetes {
         containerTemplate {
             name 'shell'
             image 'ubuntu'
            command 'sleep'
            args 'infinity'
          }
          containerTemplate{
            name 'git' 
            image 'alpine/git'
            ttyEnabled true 
            command 'cat'
          }
          containerTemplate{
            name 'maven'
            image 'maven:3.3.9-jdk-8-alpine' 
            command 'cat' 
            ttyEnabled true
          }
          containerTemplate{
            name 'docker' 
            image 'docker' 
            command 'cat' 
            ttyEnabled true
          }
          defaultContainer 'git'
      }
  }
	options {
      	skipDefaultCheckout true
				disableConcurrentBuilds();
	}
	stages {
		stage('Clean Checkout') {
			steps {
				cleanWs()
				script{
              checkoutDetails = checkout scm
              isPullRequest = verifyPullRequest();
              stage('Build App'){
              		container('maven'){
              			println("Build App via maven/node/gradle etc")
                	}
              }
              stage('Build Docker Image'){
              	println("Build Docker Image")
              }
              if(!isPullRequest){
                  stage('Push Docker image'){
                    println("Push Docker image to docker registry")
                  }
                  stage('Deploy image'){
                    println("Deploy image to k8s")
                  }
              }
          }
			}
			post {
				failure {
					println("Pipeline Failed");
				}
			}
		}
	}
	post {
		always{
			script{
          println("Always script");
			}
		}
		success {
			script{
				println("Build Succeeded");
			}
		}
		failure {
			script {
				println("Build Failed");
			}
		}
    cleanup{
      script {
          println("Cleaning up workspace");
      }
    	cleanWs();
		}
	}
}

def verifyPullRequest(){
		def pullRequestBranchName=env.branch_name;
		def pullRequestToken=pullRequestBranchName.split('-')
		if(pullRequestToken.size()==2 && pullRequestToken[0]=="PR"){
			echo("Triggered by Pull Request "+pullRequestToken[1])
			return true;
		}else{
			echo("Triggered by a commit")
			return false;
		}
}
