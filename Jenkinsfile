pipeline {
    agent {
        kubernetes {
            cloud 'kubernetes1'
            yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: jnlp
                    image: jenkins/inbound-agent:3107.v665000b_51092-15
                    tty: true
            """
        }

    tools {
        nodejs "node"
        maven "maven"
    }


    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

//         stage('Install Dependencies') {
//             steps {
//                 sh 'npm install'
//             }
//         }
        stage('Install SBOM tool') {
            steps {
                sh 'npm install -g @cyclonedx/cdxgen'
            }
        }

        stage('Generate SBOM') {
            steps {
                sh 'export FETCH_LICENSE=true && cdxgen -r -o bom.json'
                script {
                    def sbom = readFile('bom.json')
                    echo "Generated SBOM:\n$sbom"
                }
            }
        }

        stage('Upload SBOM to Dependency-Track') {
            steps {
                withCredentials([string(credentialsId: 'apikey', variable: 'X_API_KEY')]) {
                    sh """
                    curl -k -X POST "https://dt-api-jenkins-test.staging.cryptosoft.com/api/v1/bom" \
                    -H "Content-Type:multipart/form-data" \
                    -H "X-Api-Key:${X_API_KEY}" \
                    -F "autoCreate=true" \
                    -F "projectName=Jenkinsreact" \
                    -F "projectVersion=1.23" \
                    -F "bom=@bom.json"
                    """
                }
            }
        }
    }
}
}
