pipeline {
    agent any
        stages {
        stage('Build') {
            steps {
                sh 'mvn -B package'
            }
        }
        stage('SonarQube analysis') {
            steps {
                script {
                    sh "/var/jenkins_home/sonar/bin/sonar-scanner \
                        -Dsonar.projectKey=projectKey \
                        -Dsonar.projectName=projectName \
                        -Dsonar.scm.disabled=true \
                        -Dsonar.sources=. \
                        -Dsonar.language=java \
                        -Dsonar.java.binaries=./target/classes \
                        -Dsonar.sourceEncoding=UTF-8 \
                        -Dsonar.host.url=http://192.168.100.13:9000 \
                        -Dsonar.exclusions=src/test/java/****/*.java \
                        -Dsonar.login=squ_1b7bd1baab237ff86d7e54c37f9aa0f24eb941b3"
                }
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
                            nexusVersion: "nexus3",
                            protocol: "http",
                            nexusUrl: "192.168.43.188:8081",
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: "repo-nexus",
                            credentialsId: "d085a0a2-dbd8-487b-a2c1-6115c90a1e6e",
                            artifacts: [
                                    [artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging]
                                ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        }
    post{
    //    succeed {
    //        slackSend channel: '#fundamentos-de-devops', token: "slack_token", color: 'good', , message: "${custom_msg1()}", teamDomain: 'sustantivagrupo', tokenCredentialId: 'slack_token', username: 'Luis_Rivas'
    //    }
        failure{
            slackSend channel: "#fundamentos-de-devops", token: "slack_token", color: "danger", message: "${custom_msg()}", teamDomain: 'sustantivagrupo', tokenCredentialId: 'slack_token', username: 'Luis_Rivas'
        }
    }
}
    def custom_msg() {
        def JENKINS_URL= "localhost:8080"
        def JOB_NAME = env.JOB_NAME
        def BUILD_ID= env.BUILD_ID
        def JENKINS_LOG= " FAILED: Job [${env.JOB_NAME}] Logs path: ${JENKINS_URL}/job/${JOB_NAME}/${BUILD_ID}/consoleText"
        return JENKINS_LOG
    }
    // def custom_msg1() {
    //     def JENKINS_URL= "localhost:8080"
    //     def JOB_NAME = env.JOB_NAME
    //     def BUILD_ID= env.BUILD_ID
    //     def JENKINS_LOG= " SUCCEED: Job [${env.JOB_NAME}] Logs path: ${JENKINS_URL}/job/${JOB_NAME}/${BUILD_ID}/consoleText"
    //     return JENKINS_LOG
    // }
