pipeline {
    agent any
    environment {
        LOCAL_SERVER = '192.168.2.72'
    }
    tools {
        maven 'M3_8_2'
        nodejs 'NodeJs12'
    }
    stages {
        stage('Build and Analize') {
            when {
                anyOf {
                    changeset "*microservicio-service/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
            steps {
                dir('microservicio-service/'){
                    echo 'Execute Maven and Analizing with SonarServer'
                    // withSonarQubeEnv('SonarServer') {
                    sh "mvn clean package -DskipTests \
                        -Dsonar.projectKey=21_MyCompany_Microservice \
                        -Dsonar.projectName=21_MyCompany_Microservice \
                        -Dsonar.sources=src/main \
                        -Dsonar.coverage.exclusions=**/*TO.java,**/*DO.java,**/curso/web/**/*,**/curso/persistence/**/*,**/curso/commons/**/*,**/curso/model/**/* \
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \
                        -Djacoco.output=tcpclient \
                        -Djacoco.address=127.0.0.1 \
                        -Djacoco.port=10001"
                    //}
                }
            }
        }
        stage('Build and Analize two') {
            when {
                anyOf {
                    changeset "*microservicio-service-two/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
            steps {
                dir('microservicio-service-two/'){
                    echo 'Execute Maven and Analizing with SonarServer'
                    // withSonarQubeEnv('SonarServer') {
                    sh "mvn clean package  \
                        -Dsonar.projectKey=21_MyCompany_Microservice \
                        -Dsonar.projectName=21_MyCompany_Microservice \
                        -Dsonar.sources=src/main \
                        -Dsonar.coverage.exclusions=**/*TO.java,**/*DO.java,**/curso/web/**/*,**/curso/persistence/**/*,**/curso/commons/**/*,**/curso/model/**/* \
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \
                        -Djacoco.output=tcpclient \
                        -Djacoco.address=127.0.0.1 \
                        -Djacoco.port=10001"
                    //}
                }
            }
        }
        /*stage('Quality Gate'){
            when {
                anyOf {
                    changeset "*microservicio-service/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }*/
        /*stage('Frontend') {
            when {
                anyOf {
                    changeset "*frontend/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
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
        }*/
        stage('Database') {
            when {
                anyOf {
                    changeset "*liquibase/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
			steps {
				dir('liquibase/'){
					sh '/opt/liquibase/liquibase --version'
					sh '/opt/liquibase/liquibase --changeLogFile="changesets/db.changelog-master.xml" update'
					echo 'Applying Db changes'
				}
			}
		}
        stage('Container Build') {
            when {
                anyOf {
                    changeset "*microservicio-service/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
            steps {
                dir('microservicio-service/'){
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub_id', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                        sh 'docker login -u $USERNAME -p $PASSWORD'
                        sh 'docker build -t microservicio-service .'
                    }
                }
            }
        }
        stage('Container Build two') {
            when {
                anyOf {
                    changeset "*microservicio-service-two/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
            steps {
                dir('microservicio-service-two/'){
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub_id', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                        sh 'docker login -u $USERNAME -p $PASSWORD'
                        sh 'docker build -t microservicio-service-two .'
                    }
                }
            }
        }
        stage('Zuul') {
            when {
                anyOf {
                    changeset "*ZuulBase/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
            steps {
                dir('ZuulBase/'){
                    sh 'mvn clean package'
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub_id', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                        sh 'docker login -u $USERNAME -p $PASSWORD'
                        sh 'docker build -t zuul .'
                        sh 'docker stop zuul-service || true'
                        sh 'docker run -d --rm --name zuul-service -p 8000:8000 zuul'
                    }
                }
            }
        }
        stage('Eureka') {
            when {
                anyOf {
                    changeset "*EurekaBase/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
            steps {
                dir('EurekaBase/'){
                    sh 'mvn clean package'
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub_id', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                        sh 'docker login -u $USERNAME -p $PASSWORD'
                        sh 'docker build -t eureka .'
                        sh 'docker stop eureka-service || true'
                        sh 'docker run -d --rm --name eureka-service -p 8761:8761 eureka'
                    }
                }
            }
        }
        
        /*stage('Container Push Nexus') {
            when {
                anyOf {
                    changeset "*microservicio-service/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockernexus_id', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                    sh 'docker login ${LOCAL_SERVER}:8083 -u $USERNAME -p $PASSWORD'
                    sh 'docker tag microservicio-service:latest ${LOCAL_SERVER}:8083/repository/docker-private/microservicio_nexus:dev'
                    sh 'docker push ${LOCAL_SERVER}:8083/repository/docker-private/microservicio_nexus:dev'
                }
            }
        }*/
        stage('Container Run') {
            steps {
                sh 'docker stop microservicio-one || true'
                sh 'docker run -d --rm --name microservicio-one -e SPRING_PROFILES_ACTIVE=qa microservicio-service'
            }
        }
        stage('Container Run two') {
            steps {
                sh 'docker stop microservicio-two || true'
                sh 'docker run -d --rm --name microservicio-two -e SPRING_PROFILES_ACTIVE=qa microservicio-service-two'
            }
        }
        /*stage('Testing') {
            when {
                anyOf {
                    changeset "*cypress/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
            steps {
				dir('cypress/') {
                    sh 'docker build -t cypressfront .'
                    sh 'docker run cypressfront'
					//sh 'docker run --rm --name Cypress -v /Users/javierrodriguez/Documents/Repositorios/EcosistemaJenkins/jenkins_home/workspace/Microservicio/Cypress:/e2e -w /e2e -e Cypress cypress/included:3.4.0'
				}
            }
        }*
        /*stage('tar videos') 
        {
            when {
                anyOf {
                    changeset "*cypress/cypress/videos/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
            steps 
            {
                dir('cypress/cypress/videos/') {
                    sh 'tar -cvf videos.tar .'
                    archiveArtifacts artifacts: 'videos.tar',
                    allowEmptyArchive: true
                }
            }
        }*/
        /*stage('Estress') {
            when {
                anyOf {
                    changeset "*Gatling/**"
                    expression { currentBuild.previousBuild.result != "SUCCESS"}
                }
            }
            steps {
                dir('Gatling/'){
                    sh 'mvn gatling:test'
                }
            }
            post {
                always {
                    gatlingArchive()
                }
            }
        }*/
    }
    post {
        always {
            //deleteDir()
            echo 'Always'
        }
        success {
            echo 'I succeeeded!'
        }
        failure {
            echo 'I failed :('
        }
    }
}