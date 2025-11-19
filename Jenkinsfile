// Jenkinsfile – Version finale qui marche à tous les coups
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code GitHub...'
                checkout scm
            }
        }

        stage('Build & Test (avec vrai MySQL via Testcontainers)') {
            steps {
                echo 'Démarrage des tests unitaires...'
                sh 'mvn -B clean verify'
            }
        }

        stage('Package') {
            steps {
                echo 'Génération du JAR...'
                sh 'mvn -B package -DskipTests'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, onlyIfSuccessful: true
                echo 'JAR archivé avec succès !'
            }
        }
    }

    post {
        always {
            // Nettoyage propre des conteneurs Testcontainers
            sh 'docker ps -q --filter "label=org.testcontainers=true" | xargs -r docker rm -f || true'
            
            // Publication des résultats de tests
            junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true
        }
        success {
            echo '====================================================='
            echo '     BUILD VERT !!! FELICITATIONS MAHDI !'
            echo '     Ton pipeline Spring Boot + MySQL est parfait'
            echo '====================================================='
        }
        failure {
            echo 'Build échoué – regarde les logs ci-dessus'
        }
    }
}
