pipeline {
    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean deploy -Dmaven.test.skip=true'
            }
        }

        stage('Test') {
            steps {
                sh "mvn surefire-report:report"
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'soni-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('soni-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Jar Publish') {
            steps {
                script {
                    echo "BUILD_ID: ${env.BUILD_ID}"
                    echo "GIT_COMMIT: ${env.GIT_COMMIT}"

                    def server = Artifactory.server('my-artifactory-server')
                    def properties = "buildid=${env.BUILD_ID};commitid=${env.GIT_COMMIT}"

                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "soni-local/",
                                "flat": false,
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""

                    def buildInfo = server.upload spec: uploadSpec
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                }
            }
        }
    }
}
