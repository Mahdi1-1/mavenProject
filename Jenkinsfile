pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        DOCKER_HUB_CREDENTIALS = 'dbd1711b-3842-486e-b5a9-c14f84df9324' // ID des credentials Jenkins pour Docker Hub
        DOCKER_IMAGE_NAME = 'mahdimasmoudi/student-management'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
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

        stage('Docker Build & Push') {
            steps {
                echo 'Construction et push de l’image Docker...'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS}") {
                        def customImage = docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_TAG}")
                        customImage.push()
                        customImage.push('latest')
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                def qgStatus = 'NON DÉTECTÉ'
                try {
                    def qg = waitForQualityGate(abortPipeline: false)
                    qgStatus = qg.status
                } catch (err) {
                    echo "Impossible de récupérer le Quality Gate : ${err}"
                }

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
                        
                        <p>Quality Gate : <strong>${qgStatus}</strong></p>
                        
                        <h3>Docker Image</h3>
                        <p>${DOCKER_IMAGE_NAME}:${DOCKER_TAG}</p>
                        
                        <p>Voir les détails du build : <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
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
