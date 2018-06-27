pipeline {	
	
	options {
		buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
	}
	agent any
	
	parameters { 
			choice (
				choices: 'mysql\npostgres',
				description: 'Type de base de données',
				name : 'db_type')   
			
      }
	environment {
	  		COMPOSE_PROJECT_NAME = "${db_type}_${env.BUILD_ID}"
			} 

	stages {
		
		stage ('Deploiement containers avec Docker-compose') {
			
			steps { 
				script {
				    currentBuild.displayName = "Env_TNRA_${db_type}_${BUILD_NUMBER}"
					currentBuild.description = "${db_type}" 
					
				    echo " db_type : $db_type et Workspace : $WORKSPACE "
				    sh "export DOCKER_CLIENT_TIMEOUT=120 && export COMPOSE_HTTP_TIMEOUT=120"
				    
				    checkout(
				        [$class: 'GitSCM', 
				        branches: [[name: '*/master']], 
				        doGenerateSubmoduleConfigurations: false, 
				        extensions: [[$class: 'CleanBeforeCheckout']], 
				        submoduleCfg: [], 
				        userRemoteConfigs: [[credentialsId: '5598c8ea-2bfe-4c3f-84e3-a7df84708674', 
				        url: 'https://github.com/HSnippet/LD_deploy.git']]])
				    
				    echo "$db_type"
				    
				    if ("$db_type" == 'mysql') {
				        echo "Lancement dockercompose mysql "
						sh "cd $WORKSPACE/logicaldoc_mysql && docker-compose up -d"
						} else if ("$db_type" == 'postgres') {
						echo "Lancement dockercompose postgre $WORKSPACE/logicaldoc_postgres"
						sh "cd $WORKSPACE/logicaldoc_postgres && docker-compose up -d"
						}
					
					}
					
                }
			}
	
		stage ('Qualimétrie') {			
			steps { 
				echo "Lancement de l'analyse qualimétrique"
			}
		}
	
	    stage ('Lancement de la recette des web services') {
     
			steps {
			    script {
					echo "Lancement des tests de webservices (SoapUI) "
				    sh " cd $WORKSPACE/tests_webservices && /home/workspace_G3/SoapUI-5.4.0/bin/testrunner.sh LogicalDoc-soapui-project-Groupe3.xml"
			    }
		    }
	    }
	/*		post {
				always {
					if ("$db_type" == 'mysql') {
				        echo "Arrêt containers mysql "
						sh "cd $WORKSPACE/logicaldoc_mysql && docker-compose down"
						} else if ("$db_type" == 'postgres') {
						echo "Arrêt containers postgre"
						sh "cd $WORKSPACE/logicaldoc_postgres && docker-compose down"
						}
				}
			}
	*/
		
    }
}