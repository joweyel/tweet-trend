pipeline {
    agent {
        node {
            label "maven"
        }
    }
    environment {
        PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
    }

    stages {
        stage('Clone-code') {
            steps {
                sh 'mvn clen deploy'
            }
        }
    }
}
