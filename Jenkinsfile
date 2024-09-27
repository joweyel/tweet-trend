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
                echo "-------------- BUILD STARTED --------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'  // disable unit tests in build stage
                echo "-------------- BUILD COMPLETED  --------------"
            }
        }
        stage("test") {
            steps {
            echo "-------------- UNIT TEST STARTED --------------"
            sh 'mvn surefire-report:report'                    // only run unit-tests in "test" stage
            echo "-------------- UNIT TEST COMPLETED  --------------"
            }
        }

        // stage('SonarQube analysis') {
        //     environment {
        //         // configured in Dashboard -> Manage Jenkins -> Tools
        //         scannerHome = tool 'jw-sonar-scanner'
        //     }
        //     steps {
        //         // configured in Dashboard -> Manage Jenkins -> System
        //         withSonarQubeEnv('jw-sonarqube-server') {
        //             sh "${scannerHome}/bin/sonar-scanner"
        //         }
        //     }
        // }
    }
}
