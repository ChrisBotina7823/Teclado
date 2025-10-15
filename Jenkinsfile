pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup tools') {
            steps {
                sh '''
                # Instalar sshpass y nodejs
                apt-get update || true
                which sshpass || apt-get install -y sshpass
                if ! command -v node &> /dev/null; then
                echo "Instalando Node.js..."
                curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
                apt-get install -y nodejs
                fi
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner -X \
                                -Dsonar.projectKey=TallerJenkins \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=$SONAR_HOST_URL \
                                -Dsonar.login=$SONAR_LOGIN \
                                -Dsonar.password=$SONAR_PASSWORD \
                        """
                    }
                }
            }
        }
    
        stage('Deploy to Nginx') {
            steps {
                sh '''
                sshpass -p "$NGINX_PASSWORD" scp -o StrictHostKeyChecking=no -r \
                index.html css script.js \
                $NGINX_USER@$NGINX_HOST:$DEPLOY_PATH
                '''
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
