def registry = 'https://joweyel01.jfrog.io'  // for saving artifacts to jfrog
def imageName = 'joweyel01.jfrog.io/jweyel-docker-local/ttrend'
def version = '2.1.3'

pipeline {
    agent 
    {
        node 
        {
            label 'maven'
        }
    }
    environment 
    {
        PATH = "/opt/apache-maven-3.9.9/bin:$PATH"
        AWS_ACCESS_KEY_ID = credentials('aws_creds_tf_access_key')
        AWS_SECRET_ACCESS_KEY = credentials('aws_creds_tf_secret_key')
        AWS_REGION = "us-east-1"
    }

    stages 
    {
        stage('Clone-code') 
        {
            steps 
            {
                echo "-------------- BUILD STARTED --------------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'  // disable unit tests in build stage
                echo "-------------- BUILD COMPLETED  --------------"
            }
        }
        stage("test") 
        {
            steps 
            {
                echo "-------------- UNIT TEST STARTED --------------"
                sh 'mvn surefire-report:report'                    // only run unit-tests in "test" stage
                echo "-------------- UNIT TEST COMPLETED  --------------"
            }
        }

        // Commented out due severe problems that are not yet
        // stage('SonarQube analysis') 
        // {
        //     environment 
        //     {
        //         // configured in Dashboard -> Manage Jenkins -> Tools
        //         scannerHome = tool 'jw-sonar-scanner'
        //     }
        //     steps 
        //     {
        //         // configured in Dashboard -> Manage Jenkins -> System
        //         withSonarQubeEnv('jw-sonarqube-server') 
        //         {
        //             sh "${scannerHome}/bin/sonar-scanner"
        //         }
        //     }
        // }

        // stage("Quality Gate") 
        // {
        //     steps 
        //     {
        //         script 
        //         {
        //             timeout(time: 1, unit: 'HOURS') 
        //             {  // Just in case something goes wrong (pipeline will be kiled after timeout)
        //                 def qg = waitForQualityGate()  // Reuse `taskId` prev. colected by `withSonarQubeEnv`
        //                 if (qg.status != 'OK') 
        //                 {
        //                     error "Pipeline aborted due to quality gate failure: ${qg.status}"
        //                 }
        //             }
        //         }
        //     }
        // }

        // Saving Artifacts to JFrog
        stage("Jar Publish") 
        {
            steps 
            {
                script 
                {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"artifact-cred"
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                    def uploadSpec = """{
                    "files": [
                        {
                            "pattern": "jarstaging/(*)",
                            "target": "jweyel01-libs-release-local/{1}",
                            "flat": "false",
                            "props" : "${properties}",
                            "exclusions": [ "*.sha1", "*.md5"]
                        }
                    ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'  
                }
            }   
        }

        stage("Docker Build") 
        {
            steps 
            {
                script 
                {
                echo '<--------------- Docker Build Started --------------->'
                app = docker.build(imageName+":"+version)
                echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }
        stage("Docker Publish")
        {
            steps {
                script 
                {
                    echo '<--------------- Docker Publish Started --------------->'  
                    docker.withRegistry(registry, 'artifact-cred')
                    {
                        app.push()
                    }    
                    echo '<--------------- Docker Publish Ended --------------->'  
                }
            }
        }
        stage('Get cluster credentials') 
        {
            steps 
            {
                script 
                {
                    sh 'aws eks update-kubeconfig --region $AWS_REGION --name jw-eks-01'
                }
            }
        }
        stage("Deploy")
        {
            steps 
            {
                script 
                {
                    echo '<--------------- Helm Deploy Started --------------->'
                    sh 'helm install ttrend ttrend-0.1.0.tgz'
                    echo '<--------------- Helm deploy Ends --------------->'
                }
            }
        } 
    }
}
