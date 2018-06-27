pipeline {	
	
	options {
		buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '20'))
	}
	agent any
	
	parameters { 
			string (
				defaultValue: '1',
				description: 'Nombre d\'instances à déployer',
				name : 'nbInstances')			
    }
	  
	environment {	
			COMPOSE_PROJECT_NAME = "envMan-${env.BUILD_ID}"
	} 

	stages {
		
		stage ('Deploiement du container du reverse-proxy nginx'){
            steps {
		        build 'PL_Lancement_RevProxy'
		    }
		}
		
		stage ('Deploiement de l\'application LogicalDoc et de sa base de données') {
			
			steps { 
				script {
				    currentBuild.displayName = "Env_recette_manuelle_${BUILD_NUMBER}"
					currentBuild.description = "" 
					
				    echo "Workspace : $WORKSPACE - Nombre d'environnements manuels à créer : $nbInstances"
				    
				    checkout(
				        [$class: 'GitSCM', 
				        branches: [[name: '*/master']], 
				        doGenerateSubmoduleConfigurations: false, 
				        extensions: [[$class: 'CleanBeforeCheckout']], 
				        submoduleCfg: [], 
				        userRemoteConfigs: [[credentialsId: '5598c8ea-2bfe-4c3f-84e3-a7df84708674', 
				        url: 'https://github.com/HSnippet/LD_deploy.git']]])
				        
				        echo "Lancement de $nbInstances instance de clientdebian avec docker-compose"
						sh "cd $WORKSPACE/env_recette_manuelle && docker-compose up -d --scale clientdebian=$nbInstances" 
						echo "Environnement déployé et accessible à l'URL suivante : A COMPLETER " 
					}
					}
                }
	
		stage ('Transmission des coordonnées des containers à Ansible') {			
			steps { 
				 sh """echo "[clients]\n\$(docker ps -qf name='envman_*') ansible_connection=docker" > $WORKSPACE/env_recette_manuelle/ansible/hosts"""
			}
		}
	  
		stage ('Lancement de la configuration des postes par Ansible') {
	
	        steps { 
				echo "Lancement préparation postes avec Ansible"
	            sh "cd $WORKSPACE/env_recette_manuelle/ansible/ && ansible-playbook -i hosts site.yml"
	            echo "Postes configurés avec succès"
	        }	
	}
}
}
