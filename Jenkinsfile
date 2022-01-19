pipeline {
    agent any

    stages {

        stage('Code Analysis') {
            steps {
                echo 'Performing static code analysis'
                 sh 'mvn clean install sonar:sonar -DskipTests=true -Dsonar.sources=src/main -Dsonar.tests=src/test -Dsonar.projectKey=Webapp -Dsonar.projectName=Webapp -Dsonar.projectVersion=1.0 -Dsonar.login=admin -Dsonar.password=test -Dsonar.java.binaries=target/classes -Dsonar.java.test.binaries=target/test-classes'  
            }
        }

        stage('Build') {
            steps {
                echo 'Code Build and Unit Testing...'
                sh 'mvn package'
                sh "mv target/*.war target/myweb.war"
            }
        }

        stage('deploy-dev') {
            steps {
                echo 'Deploying war in Dev'
                sh 'cp target/myweb.war /var/lib/tomcat9/webapps'
		sh '	/usr/share/tomcat9/bin/catalina.sh stop'
		sh '	/usr/share/tomcat9/bin/catalina.sh start'
            }
        }

    }

    post {
        always {
            junit 'target/surefire-reports/**/*.xml'
        }
    }

}