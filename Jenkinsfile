pipeline {
    agent any
    
    tools {
        jdk 'JDK17'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout ') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/praveenece431/SpringBoot-WebApplication.git'
            }
        }
        
        stage('Code Compile') {
            steps {
                    sh "mvn compile"
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                    sh "mvn test"
            }
        }
        
        // stage('Sonarqube Analysis') {
        //     steps {
        //             withSonarQubeEnv('sonar-server') {
        //                 sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Java-WebApp \
        //                 -Dsonar.java.binaries=. \
        //                 -Dsonar.projectKey=Java-WebApp '''
    
        //         }
        //     }
        // }
        stage('Static Code Analysis') {
        environment {
          SONAR_URL = "http://104.45.197.63:9000"
         }
        steps {
          withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_AUTH_TOKEN')]) {
            sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
          }
        }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                   dependencyCheck additionalArguments: '--format HTML ./   ', odcInstallation: 'DP'
            }
        }
        
        stage('Maven Build') {
            steps {
                    sh "pwd && ls -l"
                    sh "mvn clean install -s .m2/settings.xml"
            }
        }
        
        stage('Packahe upload to Nexus') {
            steps {
                    sh "pwd && ls -l"
                    sh "mvn clean deploy -s .m2/settings.xml"
            }
        }

        stage('Docker Build & Push') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker build -t webapp ."
                            sh "docker tag webapp praveen431ece/webapp:latest"
                            sh "docker push praveen431ece/webapp:latest "
                        }
                   } 
            }
        }
        
        stage('Docker Image scan') {
            steps {
                    sh 'trivy image --format template --template "@html.tpl" -o image-scan-report.html praveen431ece/webapp:latest'
            }
        }
        
        stage('Clean up the docker image') {
            steps {
                    sh 'docker rmi webapp -f'
            }
        }

        stage('Archive artifacts') {
            steps {
                   archiveArtifacts artifacts: '**/*.html',
                   allowEmptyArchive: true,
                   fingerprint: true,
                   onlyIfSuccessful: true
            }
        }
        
    }
}