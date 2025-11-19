// Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any

    tools {
        maven 'Maven-3.9'    // nom configuré dans Jenkins → Global Tool Configuration
        jdk   'JDK-17'       // nom configuré dans Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code...'
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                echo 'Lancement des tests avec un vrai MySQL via Testcontainers...'
                sh 'mvn -B clean verify'
                // -B = mode batch (logs propres), clean verify = compile + test + package
            }
        }

        stage('Package') {
            steps {
                echo 'Génération du JAR...'
                sh 'mvn -B package -DskipTests'
                // on saute les tests ici car déjà faits dans l’étape précédente
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                echo 'JAR archivé avec succès !'
            }
        }
    }

    post {
        always {
            // Nettoyage des conteneurs Testcontainers (au cas où)
            sh 'docker ps -q --filter "label=org.testcontainers" | xargs -r docker rm -f || true'
            junit 'target/surefire-reports/*.xml'
        }
        success {
            echo 'BUILD VERT ! Bravo Mahdi !'
        }
        failure {
            echo 'Build échoué — vérifie les logs'
        }
    }
}
