pipeline {
    agent any
    triggers {
        githubPush()
    }
    environment {
        DOCKER_HUB_CREDENTIALS = 'dbd1711b-3842-486e-b5a9-c14f84df9324'
        DOCKER_IMAGE_NAME = 'mahdimasmoudi/student-management'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        KUBECONFIG_CREDENTIALS = 'kubeconfig-credentials'
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Recuperation du code depuis GitHub...'
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
                echo 'Construction et push de image Docker...'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_HUB_CREDENTIALS}") {
                        def customImage = docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_TAG}")
                        customImage.push()
                        customImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploiement sur Kubernetes...'
                script {
                    withKubeConfig([credentialsId: "${KUBECONFIG_CREDENTIALS}"]) {
                        sh '''
                            # Creer le namespace si necessaire
                            kubectl create namespace devops --dry-run=client -o yaml | kubectl apply -f -
                            
                            # Deployer MySQL
                            echo "=== Deploiement de MySQL ==="
                            kubectl apply -f mysql-deployment.yaml
                            kubectl rollout status deployment/mysql -n devops --timeout=2m
                            
                            # Mettre a jour image Spring App et deployer
                            echo "=== Deploiement de Spring App ==="
                            kubectl set image deployment/spring-app spring-app=${DOCKER_IMAGE_NAME}:${DOCKER_TAG} -n devops --record || true
                            kubectl apply -f spring-deployment.yaml
                            
                            # Patcher le deployment pour utiliser la nouvelle image
                            kubectl patch deployment spring-app -n devops -p "{\\"spec\\":{\\"template\\":{\\"spec\\":{\\"containers\\":[{\\"name\\":\\"spring-app\\",\\"image\\":\\"${DOCKER_IMAGE_NAME}:${DOCKER_TAG}\\"}]}}}}"
                            
                            # Attendre le rollout
                            kubectl rollout status deployment/spring-app -n devops --timeout=3m
                            
                            # Afficher etat du deploiement
                            echo "=== Etat du deploiement ==="
                            kubectl get pods -n devops
                            kubectl get svc -n devops
                            echo "=== Image deployee ==="
                            kubectl get deployment spring-app -n devops -o jsonpath="{.spec.template.spec.containers[0].image}"
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                def qgStatus = 'NON DETECTE'
                def k8sDeploymentStatus = 'NON DISPONIBLE'
                def deployedImage = 'NON DISPONIBLE'
                
                try {
                    withKubeConfig([credentialsId: "${KUBECONFIG_CREDENTIALS}"]) {
                        k8sDeploymentStatus = sh(
                            script: 'kubectl get deployment spring-app -n devops -o jsonpath="{.status.conditions[?(@.type==\'Available\')].status}"',
                            returnStdout: true
                        ).trim()
                        
                        deployedImage = sh(
                            script: 'kubectl get deployment spring-app -n devops -o jsonpath="{.spec.template.spec.containers[0].image}"',
                            returnStdout: true
                        ).trim()
                    }
                } catch (err) {
                    echo "Impossible de recuperer le statut Kubernetes : ${err}"
                }

                emailext (
                    subject: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${currentBuild.currentResult}",
                    body: """
                        <h2>Resultat du Build Jenkins</h2>
                        <p><strong>Statut :</strong> ${currentBuild.currentResult}</p>
                        <p><strong>Projet :</strong> ${env.JOB_NAME}</p>
                        <p><strong>Build # :</strong> ${env.BUILD_NUMBER}</p>
                        <p><strong>Duree :</strong> ${currentBuild.durationString}</p>
                        
                        <h3>SonarQube Analysis</h3>
                        <p>Lien direct vers le rapport : 
                           <a href="${env.SONAR_HOST_URL}/dashboard?id=student-management">
                           ${env.SONAR_HOST_URL}/dashboard?id=student-management</a></p>
                        <p>Quality Gate : <strong>${qgStatus}</strong></p>
                        
                        <h3>Docker Image</h3>
                        <p><strong>Image construite :</strong> ${DOCKER_IMAGE_NAME}:${DOCKER_TAG}</p>
                        <p><strong>Tag latest :</strong> ${DOCKER_IMAGE_NAME}:latest</p>
                        
                        <h3>Kubernetes Deployment</h3>
                        <p><strong>Namespace :</strong> devops</p>
                        <p><strong>Deployment Status :</strong> ${k8sDeploymentStatus}</p>
                        <p><strong>Image deployee :</strong> ${deployedImage}</p>
                        <p><strong>Deployments :</strong> mysql, spring-app</p>
                        
                        <p>Voir les details du build : <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                    """,
                    to: "mahdi.masmoudi@esprit.tn, mahdimasmoudi300@gmail.com",
                    mimeType: "text/html",
                    attachLog: true
                )
            }
        }

        success {
            echo 'BUILD ET DEPLOIEMENT REUSSIS - Tout est vert !'
        }

        failure {
            echo 'Le build ou le deploiement a echoue'
        }
    }
}


