pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    environment {
        PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
    }

    stages {
        stage('Clone-code') {
            steps {
                sh 'mvn clean deploy'
            }
        }
        stage('SonarQube analysis') {
            environment {
                // configured in Dashboard -> Manage Jenkins -> Tools
                scannerHome = tool 'jw-sonar-scanner'
            }
            steps {
                // configured in Dashboard -> Manage Jenkins -> System
                withSonarQubeEnv('jw-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    }
}
