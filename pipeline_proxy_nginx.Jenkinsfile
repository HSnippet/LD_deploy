pipeline {	
	
	options {
		buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
	}
	agent any
	
	environment {
	  		COMPOSE_PROJECT_NAME = "nginx-proxy_${env.BUILD_ID}"
			} 

	stages {
		
		stage ('Deploiement containers avec Docker-compose') {
			
			steps { 
				script {
				    currentBuild.displayName = "nginx_${BUILD_NUMBER}"
					currentBuild.description = "" 
                    sh "export DOCKER_CLIENT_TIMEOUT=120 && export COMPOSE_HTTP_TIMEOUT=120"
                    
                    def exitcodenetwork = sh(returnStatus: true, script: "docker network ls | grep nginx-proxy ; echo \$?")
                    def exitcodeps = sh(returnStatus: true, script: "docker ps | grep nginx-proxy ; echo \$?")
                    def exitcodenginxdead = sh(returnStatus: true, script: "docker ps -a | grep nginx-proxy ; echo \$?")
                    
                    checkout(
				        [$class: 'GitSCM', 
				        branches: [[name: '*/master']], 
				        doGenerateSubmoduleConfigurations: false, 
				        extensions: [[$class: 'CleanBeforeCheckout']], 
				        submoduleCfg: [], 
				        userRemoteConfigs: [[credentialsId: '5598c8ea-2bfe-4c3f-84e3-a7df84708674', 
				        url: 'https://github.com/HSnippet/LD_deploy.git']]])

                    if ( exitcodenetwork == 0){
	                echo "Il existe déjà un network nginx-proxy."
                    } else { 
	                sh "docker network create nginx-proxy"
                    }
                    
                    if ( exitcodeps != 0 && exitcodenginxdead == 0 ){
                    sh "docker rm nginx-proxy"
                    echo "Lancement dockercompose nginx"
                    sh "cd $WORKSPACE/logicaldoc_nginx && docker-compose up -d"
                    }
                    
                    if ( exitcodeps != 0 && exitcodenginxdead != 0 ){
	                echo "Lancement dockercompose nginx"
					sh "cd $WORKSPACE/logicaldoc_nginx && docker-compose up -d"
                    } 
				}	
			}		
        }
	}
}
