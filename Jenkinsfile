pipeline {
    agent any
    
    triggers {
        cron('H/5 * * * *')  // Toutes les 5 minutes
    }
    
    tools {
        jdk 'JAVA_HOME'
        maven 'maven3'
    }
    
    options {
        timeout(time: 1, unit: 'HOURS')
    }
    
    environment {
        APP_ENV = "DEV"
    }
    
    stages {
        stage('Code Checkout') {
            steps {
                echo "=====Checking out code from Git====="
                checkout scm  // Utilise checkout scm au lieu de git
            }
        }
        
        stage('Build Package') {
            steps {
                echo "=====Building Spring Boot application====="
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                echo "=====Archiving JAR file====="
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }
    }
    
    post {
        always {
            echo "======always======"
        }
        success {
            echo "=====pipeline executed successfully ====="
        }
        failure {
            echo "======pipeline execution failed======"
        }
    }
}