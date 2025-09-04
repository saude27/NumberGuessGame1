pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'SonarQube'   // SonarQube server configured in Jenkins
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'dev', url: 'https://github.com/saude27/NumberGuessGame1.git'
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
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh "/usr/share/maven/bin/mvn sonar:sonar -Dsonar.token=$SONAR_TOKEN"
                    }
                }
            }
        }

        stage('Upload to Nexus') {
    steps {
        echo 'Uploading artifact to Nexus...'
        withCredentials([
            usernamePassword(credentialsId: 'nexus-cred', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS'),
            string(credentialsId: 'nexus-url', variable: 'NEXUS_URL')
        ]) {
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
        withCredentials([
            sshUserPrivateKey(credentialsId: 'tomcat-cred', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER'),
            string(credentialsId: 'tomcat-url', variable: 'TOMCAT_IP')
        ]) {
            sh """
                # Make sure webapps directory exists
                ssh -o StrictHostKeyChecking=no -i $SSH_KEY $SSH_USER@$TOMCAT_IP 'mkdir -p /home/$SSH_USER/apache-tomcat-11.0.10/webapps'

                # Copy the WAR file
                scp -o StrictHostKeyChecking=no -i $SSH_KEY target/NumberGuessGame-1.0-SNAPSHOT.war $SSH_USER@$TOMCAT_IP:/home/$SSH_USER/apache-tomcat-11.0.10/webapps/

                # Restart Tomcat manually
                ssh -o StrictHostKeyChecking=no -i $SSH_KEY $SSH_USER@$TOMCAT_IP '/home/$SSH_USER/apache-tomcat-11.0.10/bin/shutdown.sh || true'
                ssh -o StrictHostKeyChecking=no -i $SSH_KEY $SSH_USER@$TOMCAT_IP '/home/$SSH_USER/apache-tomcat-11.0.10/bin/startup.sh'
            """
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


