pipeline {
    agent any

    parameters {
        string(name: 'FRONTEND_TAG', defaultValue: 'latest', description: 'Docker tag for frontend image')
        string(name: 'BACKEND_TAG', defaultValue: 'latest', description: 'Docker tag for backend image')
    }

    environment {
        GITHUB_CREDENTIALS = 'Github-cred'
        GITHUB_REPO = 'https://github.com/Roshanx96/wanderlust-mega-project-2.0.git'
        DOCKERHUB_USERNAME = 'roshanx'
        SONARQUBE_ENV = 'SonarQube' // Must match Jenkins SonarQube config name
        TRIVY_SEVERITY = 'CRITICAL,HIGH'
    }

    stages {

        stage('Validate Parameters') {
            steps {
                script {
                    if (!params.FRONTEND_TAG?.trim() || !params.BACKEND_TAG?.trim()) {
                        error("Both FRONTEND_TAG and BACKEND_TAG must be provided.")
                    }
                }
            }
        }

        stage("Workspace Cleanup") {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: env.GITHUB_CREDENTIALS,
                    url: env.GITHUB_REPO
            }
        }

        stage('Security Scan - Trivy') {
            steps {
                sh '''
                trivy fs ./frontend --severity ${TRIVY_SEVERITY} || true
                trivy fs ./backend --severity ${TRIVY_SEVERITY} || true
                '''
            }
        }

        stage('Security Scan - OWASP Dependency Check') {
            steps {
                sh '''
                mkdir -p dependency-check
                dependency-check.sh --project "Wanderlust" --scan . --format "ALL" --out dependency-check
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh '''
                    sonar-scanner \
                        -Dsonar.projectKey=wanderlust \
                        -Dsonar.projectName=wanderlust \
                        -Dsonar.sources=./backend \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Wait for Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Update .env Files') {
            steps {
                sh '''
                    chmod +x Automations/updatefrontendnew.sh
                    ./Automations/updatefrontendnew.sh

                    chmod +x Automations/updatebackendnew.sh
                    ./Automations/updatebackendnew.sh
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    
                    docker build -t $DOCKERHUB_USERNAME/wanderlust-frontend:${FRONTEND_TAG} ./frontend
                    docker push $DOCKERHUB_USERNAME/wanderlust-frontend:${FRONTEND_TAG}

                    docker build -t $DOCKERHUB_USERNAME/wanderlust-backend:${BACKEND_TAG} ./backend
                    docker push $DOCKERHUB_USERNAME/wanderlust-backend:${BACKEND_TAG}
                    '''
                }
            }
        }

        stage('Trigger CD Pipeline') {
            steps {
                build job: 'wanderlust-cd-pipeline', wait: false, parameters: [
                    string(name: 'FRONTEND_TAG', value: "${params.FRONTEND_TAG}"),
                    string(name: 'BACKEND_TAG', value: "${params.BACKEND_TAG}")
                ]
            }
        }
    }
    post {
        failure {
            echo "❌ The Wanderlust CI pipeline has failed. Please check Jenkins for errors."
        }
    }
}
