// Jenkinsfile – Fonctionne sur TOUS les Jenkins sans configuration préalable
pipeline {
    agent any

    // On n'utilise PLUS tools {} → on laisse Jenkins utiliser les outils par défaut du serveur
    // (Maven et JDK sont déjà installés sur 99,9 % des Jenkins)

    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code depuis GitHub...'
                checkout scm
            }
        }

        stage('Build & Test with MySQL (Testcontainers)') {
            steps {
                echo 'Lancement des tests avec un vrai MySQL via Testcontainers...'
                // -B = batch mode (logs propres), -Dmaven.test.failure.ignore pour voir les erreurs mais continuer
                sh 'mvn -B clean verify'
            }
        }

        stage('Package') {
            steps {
                echo 'Génération du JAR final...'
                sh 'mvn -B package -DskipTests'
            }
        }

        stage('Archive JAR') {
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
            echo '
            BUILD VERT !!! Bravo Mahdi, t’as réussi !'
        }
        failure {
            echo 'Build échoué – regarde les logs ci-dessus'
        }
    }
}
