pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credential')
        KUBECONFIG = credentials('k3s-credential-file')
        TRIVY_REPORT = 'rapport-trivy.json'
    }
    
    stages {
        stage('Récupération du Code') {
            steps {
                script {
                    try {
                        checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/sabehahmed/cv-generator.git', credentialsId: 'github-credential']]])
                    } catch (Exception e) {
                        error("Erreur lors de la récupération du code : ${e.getMessage()}")
                    }
                }
            }
        }
        
        stage('Vérification Docker') {
            steps {
                script {
                    try {
                        sh 'docker ps'
                    } catch (Exception e) {
                        error("Erreur lors de la vérification de Docker : ${e.getMessage()}")
                    }
                }
            }
        }
        
        stage('Création des Images Docker') {
            steps {
                script {
                    try {
                        sh 'docker build -t cv-generator:latest .'
                    } catch (Exception e) {
                        error("Erreur lors de la création de l’image Docker : ${e.getMessage()}")
                    }
                }
            }
        }
        
        stage('SonarQube Analysis') {
            tools {
                sonarQube 'SonarScanner'
            }
            steps {
                withSonarQubeEnv('MySonarQube') {
                    script {
                        try {
                            sh 'sonar-scanner'
                        } catch (Exception e) {
                            error("Erreur lors de l’analyse SonarQube : ${e.getMessage()}")
                        }
                    }
                }
            }
        }
        
        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Analyse des Vulnérabilités avec Trivy') {
            steps {
                script {
                    try {
                        // Simplified to only generate a JSON report
                        sh "trivy image --severity CRITICAL,HIGH --format json cv-generator:latest > ${TRIVY_REPORT}"
                    } catch (Exception e) {
                        echo "Erreur lors de l’analyse Trivy, mais on continue : ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Envoi des Images Docker') {
            steps {
                script {
                    try {
                        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                        sh 'docker tag cv-generator:latest sarra5/cv-generator:latest'
                        sh 'docker push sarra5/cv-generator:latest'
                    } catch (Exception e) {
                        error("Erreur lors de l’envoi de l’image Docker : ${e.getMessage()}")
                    }
                }
            }
        }
        
        stage('Déploiement dans Kubernetes') {
            steps {
                script {
                    try {
                        sh 'kubectl apply -f kubernetes/ --kubeconfig=$KUBECONFIG'
                    } catch (Exception e) {
                        error("Erreur lors du déploiement dans Kubernetes : ${e.getMessage()}")
                    }
                }
            }
        }
        
        stage('Nettoyage des Images Docker') {
            steps {
                script {
                    try {
                        sh 'docker rmi cv-generator:latest || true'
                        sh 'docker rmi sarra5/cv-generator:latest || true'
                    } catch (Exception e) {
                        echo "Erreur lors du nettoyage des images : ${e.getMessage()}"
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                try {
                    sh "docker logout"
                } catch (Exception e) {
                    echo "Erreur lors de la déconnexion de Docker Hub : ${e.getMessage()}"
                }
            }
            emailext(
                subject: "Pipeline Status: ${currentBuild.currentResult}",
                body: "Le pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER} s'est terminé avec le statut : ${currentBuild.currentResult}\n\nVérifiez les détails ici : ${env.BUILD_URL}",
                to: 'benfrajsarra5@gmail.com',
                attachmentsPattern: "**/${TRIVY_REPORT}"
            )
        }
    }
}
