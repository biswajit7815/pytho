pipeline {
    agent any 

    environment {
        DOCKERHUB_USERNAME = 'biswajit7815'
        IMAGE_NAME = 'pytho'
        IMAGE_TAG = "${BUILD_NUMBER}" //har build ka alag tag hoga for ex. 1,2,3.....
        // sonarqube scanner tool ka path load kar rahe he
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('checkout code') {
            steps {
                echo 'code checkout  done automatically via SCM'
            }
        }
        stage('OWASP security scan'){
            steps{
                echo "scanning for vulnerabilities...."
                // DP-Check oo nam he tool me dia tha.....
                // pehili bar me chalne be 22-25 mins lega because (database update hone me...)
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
            }
        }
        stage('sonarqube analysis'){
            steps{
                withSonarQubeEnv('sonar-server')
                // code ko scan karke report server par send karta he....
                sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=pytho-project -Dsonar.source=."
            }
        }

        stage('build docker image') {
            steps {
                script {
                    echo 'build docker image ......'

                    // image ke name format hoga usename/image:tag
                    sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ."

                    // ek latest tag bhi banata he taki deploy karna asan ho jaee..
                    sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('push to dockerhub') {
            steps {
                script {
                    echo 'pushing to dockerhub.....'

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker-hub-creds',
                            passwordVariable: 'DOCKER_PASS',
                            usernameVariable: 'DOCKER_USER'
                        )
                    ]) {

                        // login command
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"

                        // push command
                        sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('deploy the container') {
            steps {
                script {
                    echo "deploying application.."

                    sh "docker rm -f pytholab || true"

                    sh "docker run -d --name pytholab -p 80:80 ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                }
            }
        }
    }
    // Report generate kar lo
    post{
        always{
            // OWASP ke report graph ke rup me dikhana....
            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        }
    }
}