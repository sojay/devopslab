pipeline {
    
    agent any
 
    tools {
    maven 'Maven3'
    }
  
    stages {
        stage("Checkout") {
            steps {
            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '754392e1-65b3-45c7-a992-3c99dc45490e', url: 'https://github.com/sojay/devopslab']])
            }
        }
        
        stage("Build") {
            steps {
                sh 'mvn clean install -f MyWebApp/pom.xml'
            }
            
        }
        
        stage("Code Quality") {
            steps {
                withSonarQubeEnv('SonarQube') {
                sh 'mvn -f MyWebApp/pom.xml sonar:sonar'
                }
            }
        }
        
        stage("Nexus Upload") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: '2a1892e6-fd48-4d9f-80c9-97a469bfbf0f', groupId: 'com.dept.app', nexusUrl: 'ec2-35-183-174-89.ca-central-1.compute.amazonaws.com:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }
        }
        
        stage ('DEV Deploy') {
            steps {
                echo "deploying to DEV Env "
                deploy adapters: [tomcat9(credentialsId: '611aad3e-132f-44cc-b081-c7a4594e4277', path: '', url: 'http://ec2-35-182-33-161.ca-central-1.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage('Slack Notify') {
            steps {
                slackSend channel: 'devops', message: 'Deployment to DEV Environment was successful'
            }
        }
        
        stage ('DEV Approval') {
            steps {
                echo "Taking approval from DEV Manager for QA Deployment"
                timeout(time: 7, unit: 'DAYS') {
                input message: 'Do you want to deploy to QA?', submitter: 'admin'
                }
            }
        }
        stage ('QA Deploy') {
            steps {
                echo "deploying to QA Env "
                deploy adapters: [tomcat9(credentialsId: '611aad3e-132f-44cc-b081-c7a4594e4277', path: '', url: 'http://ec2-35-182-33-161.ca-central-1.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
            }
        }
    
    
        stage ('QA Approve') {
            steps {
                echo "Taking approval from QA manager"
                timeout(time: 7, unit: 'DAYS') {
                input message: 'Do you want to proceed to PROD?', submitter: 'admin,manager_userid'
                }
            }
        }
    
    
        stage ('Slack Notification for QA Deploy') {
            steps {
                echo "deployed to QA Env successfully"
                slackSend(channel:'devops', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
        }  
    }
}
