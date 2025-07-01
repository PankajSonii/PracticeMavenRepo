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
                scannerHome = tool 'pankaj-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('pankaj-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Jar Publish') {
            steps {
                script {
                    echo "BUILD_ID: ${env.BUILD_ID}"
                    echo "GIT_COMMIT: ${env.GIT_COMMIT}"

                    // Use the configured Artifactory server
                    def server = Artifactory.server('my-artifactory-server')

                    def properties = "buildid=${env.BUILD_ID},commitid=${env.GIT_COMMIT}"

                    def uploadSpec = """
                    {
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "soni-local/",
                                "flat": false,
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }
                    """

                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                }
            }
        }
    }
}
