pipeline{ 
	environment{
			registry = "subugirish/docker-test"
			registryCredential = 'dockerhub'
			dockerImage = ''
			}
	agent any
	stages{
		stage('github checkout') {
			steps {
			checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'a9f4a320-4ad5-4f18-8fe8-6faa01f60c18', url: 'https://github.com/subbanna/bwce.git']]])
            }
		}
            
		stage('Build ear & docker Image'){
            steps{
			withMaven(
				// Maven installation declared in the Jenkins "Global Tool Configuration"

				maven: 'Maven',

				// Maven settings.xml file defined with the Jenkins Config File Provider Plugin

				// Maven settings and global settings can also be defined in Jenkins Global Tools Configuration

               mavenLocalRepo: '.repository'){

					sh "mvn -f SampleRest.application.parent/pom.xml clean install  docker:build -D docker.property.file=docker-dev.properties"

				}
            }

		}
		stage('Code Coverage Step'){
			steps{
                 withMaven(

				// Maven installation declared in the Jenkins "Global Tool Configuration"

				maven: 'Maven',

				// Maven settings.xml file defined with the Jenkins Config File Provider Plugin

				// Maven settings and global settings can also be defined in Jenkins Global Tools Configuration

               mavenLocalRepo: '.repository') {

			sh "mvn -f  SampleRest.application.parent/pom.xml -D sonar.host.url=http://54.254.140.127:9000 org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar"
			}
			}
		}
		stage('Start the Container') {
            steps{
				sh 'docker run -d subugirish/samplerest:latest'
			}
		}
		stage ("wait_prior_starting_smoke_testing") {
			steps {

				echo 'Waiting 5 minutes for deployment to complete prior starting smoke testing'
				sleep 180 // seconds
			}

		}
        stage('Unit testing'){
            steps {
			
				withMaven(maven: 'Maven', mavenSettingsFilePath: '/home/wiproosi/maven/apache-maven-3.6.0/conf/settings.xml', options: [junitPublisher(healthScaleFactor: 1.0)], tempBinDir: '') {

						// some block
						sh "mvn -f soapui/pom.xml -B test"
				}
            }   
        }  
		stage('Publish Image') {
			steps {
			script {
				docker.withRegistry( '', registryCredential ) {
				sh '''
				docker push subugirish/samplerest:latest
				'''
                }
			}
			}
		}  

	}

    post {

        failure {    // notify users when the Pipeline fails

			emailext attachLog: true, body: 'PLEASE CHECK THE LOG FILE FOR Errors', subject: '$BUILD_STATUS', to: 'subbanna.girish@wipro.com'
        }          
		success {    // notify users when the Pipelineis successful

			emailext attachLog: true, body: 'PLEASE CHECK THE LOG FILE FOR Details', subject: '$BUILD_STATUS', to: 'subbanna.girish@wipro.com'

		}
    }
}
