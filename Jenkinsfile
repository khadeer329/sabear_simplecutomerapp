pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'feature-1.1',
                    url: 'https://github.com/betawins/sabear_simplecutomerapp.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonarqubeScanner'

                    withSonarQubeEnv('Sonarqube') {
                        sh """
                            "${scannerHome}/bin/sonar-scanner" \
                              -Dsonar.projectKey=Ncodeit \
                              -Dsonar.projectName=Ncodeit \
                              -Dsonar.projectVersion=2.0 \
                              -Dsonar.sources=src \
                              -Dsonar.java.binaries=target/classes \
                              -Dsonar.junit.reportPaths=target/surefire-reports
                        """
                    }
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'

                    def artifacts = findFiles(
                        glob: "target/*.${pom.packaging}"
                    )

                    if (artifacts.length == 0) {
                        error "No ${pom.packaging} artifact found in target/"
                    }

                    def artifactPath = artifacts[0].path

                    if (!fileExists(artifactPath)) {
                        error "Artifact not found: ${artifactPath}"
                    }

                    echo "Uploading ${artifactPath}"
                    echo "Group: ${pom.groupId}"
                    echo "Artifact: ${pom.artifactId}"
                    echo "Version: ${pom.version}"

                    nexusArtifactUploader(
                        nexusVersion: env.NEXUS_VERSION,
                        protocol: env.NEXUS_PROTOCOL,
                        nexusUrl: env.NEXUS_URL,
                        groupId: pom.groupId,
                        artifactId: pom.artifactId,
                        version: pom.version,
                        repository: env.NEXUS_REPOSITORY,
                        credentialsId: env.NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [
                                artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging
                            ],
                            [
                                artifactId: pom.artifactId,
                                classifier: '',
                                file: 'pom.xml',
                                type: 'pom'
                            ]
                        ]
                    )
                }
            }
        }
    }

    post {
        success {
            echo 'Maven build, SonarQube analysis, and Nexus upload completed.'
        }

        failure {
            echo 'Pipeline failed. Check the first error in Console Output.'
        }
    }
}
