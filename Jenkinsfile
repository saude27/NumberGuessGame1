pipeline {
    agent any

    environment {
        GIT_CREDENTIALS    = 'github-token'         // GitHub token (if repo is private)
        SONARQUBE_ENV      = 'SonarQube'            // SonarQube server configured in Jenkins
        SONAR_TOKEN        = 'sonar-token'          // Secret Text (token stored in Jenkins)
        TOMCAT_CREDENTIALS = 'tomcat-cred'   // SSH Username with private key
        TOMCAT_IP          = 'tomcat-url'            // Secret Text (IP address)
        NEXUS_CREDENTIALS  = 'nexus-cred'    // Username + Password
        NEXUS_URL          = 'nexus-url'            // Secret Text (Nexus repo URL)
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'dev',
                    url: 'https://github.com/saude27/NumberGuessGame1.git',
                    credentialsId: "${GIT_CREDENTIALS}"
            }
        }

        stage('Build & Test') {
            steps {
                sh '/usr/share/maven/bin/mvn clean package -DskipTests=false'
                junit '**/target/surefire-reports/*.xml'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    withCredentials([string(credentialsId: "${SONAR_TOKEN}", variable: 'SONAR_TOKEN')]) {
                        sh "/usr/share/maven/bin/mvn sonar:sonar -Dsonar.token=$SONAR_TOKEN"
                    }
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                echo 'Uploading artifact to Nexus...'
                withCredentials([usernamePassword(credentialsId: "${NEXUS_CREDENTIALS}",
                                                 usernameVariable: 'NEXUS_USER',
                                                 passwordVariable: 'NEXUS_PASS'),
                                 string(credentialsId: "${NEXUS_URL}", variable: 'NEXUS_URL')]) {
                    sh """
                        /usr/share/maven/bin/mvn deploy \
                            -DskipTests=true \
                            -Dnexus.url=$NEXUS_URL \
                            -Dnexus.username=$NEXUS_USER \
                            -Dnexus.password=$NEXUS_PASS
                    """
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo 'Deploying WAR to Tomcat...'
                withCredentials([sshUserPrivateKey(credentialsId: "${TOMCAT_CREDENTIALS}",
                                                  keyFileVariable: 'SSH_KEY',
                                                  usernameVariable: 'SSH_USER'),
                                 string(credentialsId: "${TOMCAT_IP}", variable: 'TOMCAT_IP')]) {
                    sh """
                        scp -i $SSH_KEY target/NumberGuessGame-1.0-SNAPSHOT.war $SSH_USER@$TOMCAT_IP:/opt/tomcat/webapps/
                        ssh -i $SSH_KEY $SSH_USER@$TOMCAT_IP 'sudo systemctl restart tomcat'
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}

