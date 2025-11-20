pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
        choice(name: 'DEPLOY_ENV', choices: ['staging', 'production'], description: 'Deployment environment')
    }

    environment {
        GIT_COMMIT_HASH = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        BUILD_VERSION = "${env.BUILD_NUMBER}-${env.GIT_COMMIT_HASH}"
        IMAGE_NAME = "jenkinscicdspringlab:${BUILD_VERSION}"
    }

    stage('Build') {
    steps {
        sh 'chmod +x gradlew'
        sh './gradlew build'
    }
}
    stages {

        stage('Checkout') {
            steps {
                echo "Cloning repository branch: ${params.BRANCH}"
                git branch: "${params.BRANCH}", url: 'https://github.com/mohammedgassamma/cicdjenkins.git'
            }
        }

        stage('Build') {
            steps {
                echo "Building the project with Maven..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Parallel Testing') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                        junit '**/target/surefire-reports/*.xml'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'mvn verify -DskipUnitTests'
                        junit '**/target/failsafe-reports/*.xml'
                    }
                }
            }
        }

        stage('Docker Image Build') {
            steps {
                echo "Building Docker image: ${IMAGE_NAME}"
                script {
                    docker.build("${IMAGE_NAME}")
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo "Archiving artifacts..."
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('Deployment Simulation') {
            when {
                expression { params.DEPLOY_ENV == 'staging' }
            }
            steps {
                echo "Simulating deployment..."
                sh "docker run --rm -d -p 8081:8080 ${IMAGE_NAME}"
                echo "Application deployed successfully on local Docker network."
            }
        }
    }

    post {
        success {
            echo "✅ Build ${env.BUILD_NUMBER} succeeded!"
            emailext(
                subject: "✔️ SUCCESS: Build ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Le build a réussi !</p>
                         <p><b>Job :</b> ${env.JOB_NAME}</p>
                         <p><b>Build :</b> #${env.BUILD_NUMBER}</p>
                         <p><b>Branch :</b> ${params.BRANCH}</p>
                         <p><b>Image Docker :</b> ${IMAGE_NAME}</p>""",
                to: "momog9997@gmail.com"
            )
        }
        failure {
            echo "❌ Build ${env.BUILD_NUMBER} failed!"
            emailext(
                subject: "❌ FAILURE: Build ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<p>Le build a échoué ❗</p>
                         <p><b>Job :</b> ${env.JOB_NAME}</p>
                         <p><b>Build :</b> #${env.BUILD_NUMBER}</p>
                         <p><b>Branch :</b> ${params.BRANCH}</p>""",
                to: "momog9997@gmail.com"
            )
        }
    }
}
