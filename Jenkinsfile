pipeline {
    agent any
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/EmAdd9/Demo-React-To-Do-App.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Demo-React-To-Do-App\
                    -Dsonar.projectKey=Demo-React-To-Do-App \
                    -Dsonar.sources=. '''
                    
                }
                
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'owasp'
                     dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '922378b9-2188-4bee-ae19-7a1407df0a66', toolName: 'docker') {
                        sh "docker build -t app -f backend/Dockerfile ."
                        sh "docker tag app username/react-to-do-app:latest"
                        sh "docker push username/react-to-do-app:latest"
                    }
                }
            }
        }
        stage('Trivy Docker Scan') {
            steps {
                sh "trivy image username/react-to-do-app:latest"
            }
        }
        stage('Docker Deployment') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '922378b9-2188-4bee-ae19-7a1407df0a66', toolName: 'docker') {
                        sh "docker run -d --name reactapp -p 4000:4000 username/react-to-do-app:latest"
                    }
                }
            }
        }
    }
}
