pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
        choice(name: 'DEPLOY_ENV', choices: ['staging', 'production'], description: 'Deployment environment')
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning repository branch: ${params.BRANCH}"
                git branch: "${params.BRANCH}", url: 'https://github.com/mohammedgassamma/cicdjenkins.git'
            }
        }


    environment {
        GIT_COMMIT_HASH = ''
        BUILD_VERSION   = ''
        IMAGE_NAME      = ''
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning repository branch: ${params.BRANCH}"
                git branch: "${params.BRANCH}", url: 'https://github.com/mohammedgassamma/cicdjenkins.git'
            }
        }

        stage('Fix Permissions') {
            steps {
                sh "chmod +x mvnw || true"
                sh "chmod +x gradlew || true"
            }
        }

        stage('Compute Build Metadata') {
            steps {
                script {
                    env.GIT_COMMIT_HASH = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    env.BUILD_VERSION   = "${env.BUILD_NUMBER}-${env.GIT_COMMIT_HASH}"
                    env.IMAGE_NAME      = "jenkinscicdspringlab:${env.BUILD_VERSION}"

                    echo "Commit: ${env.GIT_COMMIT_HASH}"
                    echo "Image:  ${env.IMAGE_NAME}"
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    if (fileExists('mvnw')) {
                        echo "Building project with Maven wrapper..."
                        sh "./mvnw clean package -DskipTests"
                    } else if (fileExists('gradlew')) {
                        echo "Building project with Gradle wrapper..."
                        sh "./gradlew build"
                    } else {
                        error "❌ No Maven or Gradle build file found."
                    }
                }
            }
        }

        stage('Parallel Testing') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh "mvn test || ./gradlew test || true"
                        junit '**/target/surefire-reports/*.xml'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh "mvn verify -DskipUnitTests || true"
                        junit '**/target/failsafe-reports/*.xml'
                    }
                }
            }
        }

        stage('Docker Image Build') {
            steps {
                echo "Building Docker image: ${env.IMAGE_NAME}"
                script {
                    docker.build(env.IMAGE_NAME)
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        stage('Deployment Simulation') {
            when {
                expression { params.DEPLOY_ENV == 'staging' }
            }
            steps {
                echo "Simulating deployment..."
                sh "docker run --rm -d -p 8081:8080 ${env.IMAGE_NAME}"
                echo "App deployed locally on Docker!"
            }
        }
    }

    post {
        success {
            echo "✔️ Build ${env.BUILD_NUMBER} succeeded!"
        }
        failure {
            echo "❌ Build ${env.BUILD_NUMBER} failed!"
        }
    }
}
