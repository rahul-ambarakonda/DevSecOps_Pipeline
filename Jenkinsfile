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
        // Run Maven commands using Windows CMD
        bat "\"${mvnHome}\\bin\\mvn\" -Dmaven.test.failure.ignore clean package"
    }

    stage('Publish Tests Results') {
        parallel(
            publishJunitTestsResultsToJenkins: {
                echo "Publishing JUnit test results"
                junit '**/target/surefire-reports/TEST-*.xml'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        )
    }

    stage('Build Docker Image') {
        echo "Preparing JAR for Docker build"
        bat "mkdir data"
        bat "move target\\hello*.jar data\\"

        echo "Building Docker Image using Docker Named Pipe"

        // Explicitly set the Docker socket for Windows named pipe
        docker.withServer('npipe:////./pipe/docker_engine') {
            docker.build("${dockerImageName}")}
    }

    stage('Deploy Docker Image') {
        echo "Docker Image Tag Name: ${dockerImageTag}"

        // Deploy Docker image to a repository
        bat "docker login -u admin -p admin123 ${dockerRepoUrl}"
        bat "docker tag ${dockerImageName} ${dockerImageTag}"
        bat "docker push ${dockerImageTag}"
    }
}

