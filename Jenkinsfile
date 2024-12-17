node {
    // Reference the Maven tool
    def mvnHome = tool name: 'maven-3.8.8', type: 'maven'

    def dockerRepoUrl = "localhost:8083"
    def dockerImageName = "hello-world-java"
    def dockerImageTag = "${dockerRepoUrl}/${dockerImageName}:${env.BUILD_NUMBER}"

    stage('Clone Repo') { 
        git branch: 'main', url: 'https://github.com/rahul-ambarakonda/DevSecOps_Pipeline.git'
    }

    stage('Build Project') {
        bat "\"${mvnHome}\\bin\\mvn\" -Dmaven.test.failure.ignore clean package"
    }

    stage('Publish Test Results') {
        echo "Publishing JUnit test results"
        parallel(
            publishJunitResults: {
                junit '**/target/surefire-reports/TEST-*.xml'
            },
            archiveArtifactsStep: {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        )
    }

    stage('Build Docker Image') {
        echo "Preparing JAR for Docker build"
        bat "mkdir data"
        bat "move target\\hello*.jar data\\"

        echo "Building Docker Image"
        script {
            docker.withServer('npipe:////./pipe/docker_engine') {
                def image = docker.build("${dockerImageName}")
                image.tag("${dockerImageTag}")
            }
        }
    }

    stage('Push Docker Image') {
        echo "Pushing Docker Image to Repository"
        script {
            docker.withServer('npipe:////./pipe/docker_engine') {
                docker.withRegistry("http://${dockerRepoUrl}", "admin:admin123") {
                    def image = docker.image("${dockerImageTag}")
                    image.push()
                }
            }
        }
    }
}

