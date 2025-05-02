pipeline {
    agent any

    environment {
        REGISTRY = "localhost:5000"
        IMAGE_NAME = "cv-generator"
    }

    stages {
        stage('Cloner le repo') {
            steps {
                git 'https://github.com/sabehahmed/cv-generator.git'
            }
        }

        stage('Installer les dépendances') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Next.js') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Construire l\'image Docker') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Push dans le registry local') {
            steps {
                sh 'docker tag $IMAGE_NAME:latest $REGISTRY/$IMAGE_NAME:latest'
                sh 'docker push $REGISTRY/$IMAGE_NAME:latest'
            }
        }

        stage('Déployer container') {
            steps {
                sh 'docker stop $IMAGE_NAME || true'
                sh 'docker rm $IMAGE_NAME || true'
                sh 'docker run -d --name $IMAGE_NAME -p 3000:3000 $REGISTRY/$IMAGE_NAME:latest'
            }
        }
    }
}
