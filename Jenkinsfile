pipeline {
    agent any
    environment {
        //Can declare different enviornment variables to be used specifically within this pipeline
        DOCKER_IMAGE_NAME = "faveme/project1"
        MAVEN_IMAGE_NAME = "log-aggregation-demo:0.0.1-SNAPSHOT"
    }
    stages {

        stage('Build') {
            steps {
                sh 'chmod +x mvnw && ./mvnw spring-boot:build-image'
                sh 'docker tag $MAVEN_IMAGE_NAME $DOCKER_IMAGE_NAME'
                script {
                    app = docker.image(DOCKER_IMAGE_NAME)
                }
            }
        }
        stage ('Sonar Quality Analysis'){
            steps {
                withdSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonarcloud') {
                    sh 'mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar'
                }
            }
        }
        stage('Wait for Quality Gate') {
            steps {
            timeout(time: 30, unit: 'MINUTES') {
              def qualitygate = waitForQualityGate abortpipeline: true   
                }
            }
        }
        stage('Push Docker Image') {
            // when {
            //     //only executes the Docker Push stage if we are on the master branch
            //     branch 'master'
            // }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-jenkins-token') {
                        app.push( "latest")//these are the image tags
                        //app.push("${env.BUILD_NUMBER}")
                        app.push("${env.GIT_COMMIT}")
                    
                    }
                }
            }
        }
    }
    // post {
    //     always {
    //         // can use the previously created qualitygate variable to perhaps include results of as part of discordsend insturctions
    //         //might use another "script" scope
    //     }
    // }
}