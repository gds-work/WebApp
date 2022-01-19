pipeline {
    agent any

    stages {

        stage('Code Analysis') {
            steps {
                echo 'Performing static code analysis'
                 sh 'mvn clean install sonar:sonar -DskipTests=true -Dsonar.sources=src/main -Dsonar.tests=src/test -Dsonar.projectVersion=1.0 -Dsonar.login=admin -Dsonar.password=admin -Dsonar.java.binaries=target/classes -Dsonar.java.test.binaries=target/test-classes'  
            }
        }

        stage('Build') {
            steps {
                echo 'Code Build and Unit Testing...'
                sh 'mvn package'
                sh "mv target/*.war target/myweb.war"
            }
        }

        stage('deploy-to-test') {
            steps {
                echo 'Deploying war in Test'
                sh 'cp target/myweb.war /opt/tomcat/webapps'
		sh 'sleep 100'
            }
        }

        stage('FunctionalTest') {
            steps {
                echo 'Running in Functional Test'
                sh 'mvn -f functionaltest/ compile test'
            }
        }

        stage('Release Package') {
            steps {
                echo 'Releasing Package to Nexus repo'
            }
        }

    }

    post {
        always {
            junit 'target/surefire-reports/**/*.xml'
            junit 'Acceptancetest/target/surefire-reports/**/*.xml'
        }
    }

}