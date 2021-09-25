pipeline {
    agent any
    environment {
        LOCAL_SERVER = '192.168.96.1'
    }
    tools {
        maven 'M3_8_2'
        nodejs 'NodeJs12'
    }
    stages {
        stage('Build and Analize') {
            steps {
                dir('microservicio-service/'){
                    echo 'Execute Maven and Analizing with SonarServer'
                    withSonarQubeEnv('SonarServer') {
                        //sh "mvn clean package sonar:sonar \
                        sh "mvn clean package dependency-check:check sonar:sonar \
                            -Dsonar.projectKey=21_MyCompany_Microservice \
                            -Dsonar.projectName=21_MyCompany_Microservice \
                            -Dsonar.sources=src/main \
                            -Dsonar.coverage.exclusions=**/*TO.java,**/*DO.java,**/curso/web/**/*,**/curso/persistence/**/*,**/curso/commons/**/*,**/curso/model/**/* \
                            -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \
                            -Djacoco.output=tcpclient \
                            -Djacoco.address=127.0.0.1 \
                            -Djacoco.port=10001"
                    }
                }
            }
        }
        stage('Quality Gate'){
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage('Frontend') {
            steps {
                echo 'Building Frontend'
                dir('frontend/'){
                    sh 'npm install'
                    sh 'npm run build'
                    sh 'docker stop frontend-one || true'
                    sh "docker build -t frontend-web ."
                    sh 'docker run -d --rm --name frontend-one -p 8010:80 frontend-web'
                }
            }
        }

        stage('Database') {
            steps {
                dir('liquibase/'){
                    sh '/opt/liquibase/liquibase --version'
                    sh '/opt/liquibase/liquibase --changeLogFile="changesets/db.changelog-master.xml" update'
                    echo 'Applying Db changes'
                }
            }
        }
        stage('Container Build') {
            steps {
                dir('microservicio-service/'){
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub_id  ', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                        sh 'docker login -u $USERNAME -p $PASSWORD'
                        sh 'docker build -t microservicio-service .'
                        sh 'docker push luisangelhm88/microservicio-hub:dev'
                    }
                }
            }
        }
        stage('Container Push Nexus') {
            steps {
                dir('microservicio-service/'){
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockernexus_id', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                        sh 'docker login ${LOCAL_SERVER}:8083 -u $USERNAME -p $PASSWORD'
                        sh 'docker tag microservicio-service:latest ${LOCAL_SERVER}:8083/repository/docker-private/microservicio_nexus:dev'
                    }
                }
            }
        }
        stage('Container Run') {
            steps {
                sh 'docker stop microservicio-one || true'
                //sh 'docker run -d --rm --name microservicio-one -p 8090:8090 microservicio-service'
                sh 'docker run -d --rm --name microservicio-one -e SPRING_PROFILES_ACTIVE=qa-p 8090:8090 192.168.1.133:8083/repository/docker-private/microservicio_nexus:dev'
            }
        }
    }
}