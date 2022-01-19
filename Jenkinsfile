pipeline {
    agent any

    tools {
        maven "Maven"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "localhost:8081"
        NEXUS_REPOSITORY = "maven-nexus-repo"
        NEXUS_CREDENTIAL_ID = "nexus"
    }

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

       stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }

    }

    post {
        always {
            junit 'target/surefire-reports/**/*.xml'
            junit 'functionaltest/target/surefire-reports/**/*.xml'
        }
    }

}