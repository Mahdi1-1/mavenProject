pipeline {
    agent any
    triggers {
        githubPush()
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code depuis GitHub...'
                checkout scm
            }
        }

        stage('Build sans tests') {
            steps {
                echo 'Compilation avec Maven...'
                sh 'mvn -B clean install -DskipTests'
            }
        }

        stage('Build & SonarQube analysis') {
    steps {
        echo 'Analyse SonarQube en cours...'
        withSonarQubeEnv('My SonarQube Server') {
            sh '''
                mvn -B clean verify sonar:sonar \
                    -Dspring.datasource.url="jdbc:mysql://my-mysql:3306/studentdb?createDatabaseIfNotExist=true&allowPublicKeyRetrieval=true&useSSL=false&serverTimezone=UTC" \
                    -Dspring.datasource.username=root \
                    -Dspring.datasource.password= \
                    -Dspring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
            '''
        }
    }
}

        stage('Quality Gate') {
        steps {
            timeout(time: 3, unit: 'MINUTES') {
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Quality Gate failed: ${qg.status}"
                        }
                    }
                }    
            }
        }

        stage('Archive JAR') {
            steps {
                echo 'Archivage du fichier JAR...'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: false
            }
        }
    }

    post {
        always {
        // Envoi du mail avec le lien SonarQube + statut Quality Gate
        emailext (
            subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${currentBuild.currentResult}",
            body: """
                <h2>Résultat du Build Jenkins</h2>
                <p><strong>Statut :</strong> ${currentBuild.currentResult}</p>
                <p><strong>Projet :</strong> ${env.JOB_NAME}</p>
                <p><strong>Build # :</strong> ${env.BUILD_NUMBER}</p>
                <p><strong>Durée :</strong> ${currentBuild.durationString}</p>
                
                <h3>SonarQube Analysis</h3>
                <p>Lien direct vers le rapport : 
                   <a href="${env.SONAR_HOST_URL}/dashboard?id=student-management">
                   ${env.SONAR_HOST_URL}/dashboard?id=student-management</a></p>
                
                <p>Quality Gate : <strong>${currentBuild.currentResult == 'SUCCESS' ? 'PASSED' : 'FAILED'}</strong></p>
                
                <p>Voir les détails du build : <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
            """,
            to: "mahdi.masmoudi@esprit.tn, mahdimasmoudi300@gmail.com",  // mets les mails que tu veux
            mimeType: "text/html",
            attachLog: true
        )
    }
        success {
            echo ' BUILD RÉUSSI – Tout est vert !'
        }
        failure {
            echo 'Le build a échoué'
        }
    }
}



