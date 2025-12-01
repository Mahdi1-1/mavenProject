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

        stage('Archive JAR') {
            steps {
                echo 'Archivage du fichier JAR...'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: false
            }
        }
    }

    post {
    always {
        script {
            def qgStatus = 'NON DÉTECTÉ'
            try {
                timeout(time: 5, unit: 'MINUTES') {
                    def qg = waitForQualityGate(abortPipeline: false)
                    qgStatus = qg.status
                }
            } catch (err) {
                echo "Impossible de récupérer le Quality Gate : ${err}"
            }

            emailext (
                subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${currentBuild.currentResult}",
                body: """
                    <h2>Résultat du Build Jenkins</h2>
                    <p><strong>Statut :</strong> ${currentBuild.currentResult}</p>
                    <p><strong>Quality Gate :</strong> <strong>${qgStatus}</strong></p>
                    <p>Lien SonarQube : <a href="${env.SONAR_HOST_URL}/dashboard?id=student-management">${env.SONAR_HOST_URL}/dashboard?id=student-management</a></p>
                """,
                to: "mahdi.masmoudi@esprit.tn, mahdimasmoudi300@gmail.com",
                mimeType: "text/html",
                attachLog: true
            )
        }
    }

        success {
            echo 'BUILD RÉUSSI – Tout est vert !'
        }

        failure {
            echo 'Le build a échoué'
        }
    }
}

