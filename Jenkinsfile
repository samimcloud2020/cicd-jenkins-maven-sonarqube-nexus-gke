pipeline {
    
	agent any
	
	tools {
        maven "sam-maven"
        jdk "sam-jdk"
    }
	
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "10.128.0.32:8081"
        NEXUS_REPOSITORY = "vprofile-release"
	    NEXUS_REPO_ID = "vprofile-release"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
        ARTVERSION = "${env.BUILD_ID}"
		PROJECT_ID = 'genuine-fold-316617'
        CLUSTER_NAME = 'stagging'
        CLUSTER_NAME1 = "production"
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'multi-k8s'
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

	    stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

	    stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
	
/*
        stage('CODE ANALYSIS with SONARQUBE') {
          
		    environment {
                scannerHome = tool 'sonarscanner4'
          }

            steps {
                withSonarQubeEnv('sonar-pro') {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
          }
        }
*/
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTVERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } 
			        else {
				        error "*** File: ${artifactPath} , cloud not found";
			        }
                }
            }
        }
    
        stage ('Pull War file from Nexus') {
        	steps {
			    script{
                    withCredentials([usernameColonPassword(credentialsId: 'nexuslogin', variable: 'NEXUS_CREDENTIALS_ID')]) {
                    sh  'curl -u ${NEXUS_CREDENTIALS_ID} -o nexus.war "http://104.155.163.202:8081/repository/vpro-maven-group/com/crunchify/MavenTutorial/15/MavenTutorial-"${ARTVERSION}".war"'
	       	        }
		        }
            }
        }
	    stage (" Make DockerFile"){
		    steps {
			    script {
				     withCredentials([usernameColonPassword(credentialsId: 'nexuslogin', variable: 'NEXUS_CREDENTIALS_ID')]) {
				     sh "rm -rf p1"
				     sh "mkdir p1"
				     sh "cd p1"
				     sh  'curl -u ${NEXUS_CREDENTIALS_ID} -o nexus.war "http://104.155.163.202:8081/repository/vpro-maven-group/com/crunchify/MavenTutorial/15/MavenTutorial-"${ARTVERSION}".war"'
			         sh "touch Dockerfile"
					 sh " echo FROM tomcat >> Dockerfile"
					 sh " echo ADD nexus.war /usr/local/tomcat/webapps >> Dockerfile"
					 sh " echo CMD catalina.sh run >> Dockerfile"
					 sh " echo EXPOSE 8080 >> Dockerfile"    
					 }
				}
			}
		}
		stage("Build image") {
            steps {
                script {
                    myapp = docker.build("samimbsnl/cicd:${env.BUILD_ID}")
                }
            }
        }
        stage("Push image to Dockerhub") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerid') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }        
        stage("Pull image to Dockerhub") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerid') {
                        sh " docker pull samimbsnl/cicd:${env.BUILD_ID}"
                        
                    }
                }
            }
        }
        stage("Tag image to Dockerhub") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerid') {
                        sh " docker tag samimbsnl/cicd:${env.BUILD_ID} gcr.io/genuine-fold-316617/cicd:${env.BUILD_ID}"
                        
                    }
                }
            }
        }    
        stage("psuh image to gcr") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerid') {
                        
                        sh "gcloud iam service-accounts keys create keyfile.json --iam-account jenkinscicd@genuine-fold-316617.iam.gserviceaccount.com"
                        sh " gcloud auth activate-service-account jenkinscicd@genuine-fold-316617.iam.gserviceaccount.com --key-file=keyfile.json"
                        sh " gcloud auth print-access-token | docker login -u oauth2accesstoken --password-stdin https://gcr.io"
                        sh " docker push gcr.io/genuine-fold-316617/cicd:${env.BUILD_ID}"
                    }
                }
            }
        }  
	stage('Deploy to GKE Stagging') {
            steps{
                sh "sed -i 's/cicd:latest/cicd:${env.BUILD_ID}/g' deploy.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deploy.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }
	    
       stage('Wait for SRE Approval') {
            steps{
                timeout(time:12, unit:'HOURS') {
                    input message:'Approve deployment?'
                }
            }
        }
	stage('Deploy to GKE Production') {
            steps{
                sh "sed -i 's/cicd:latest/cicd:${env.BUILD_ID}/g' deploy.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME1, location: env.LOCATION, manifestPattern: 'deploy.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }    
			       		     

}
}
