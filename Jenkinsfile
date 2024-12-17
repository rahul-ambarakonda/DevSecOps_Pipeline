node {
    // Maven tool reference
    def mvnHome = tool 'maven-3.8.8'

    // Docker repository configuration
    def dockerRepoUrl = "localhost:8083"
    def dockerImageName = "hello-world-java"
    def dockerImageTag = "${dockerRepoUrl}/${dockerImageName}:${env.BUILD_NUMBER}"
    
    def dockerImage

    stage('Clone Repo') { 
        // Clone the repository
        git 'https://github.com/rahul-ambarakonda/DevSecOps_Pipeline.git'
        mvnHome = tool 'maven-3.8.8'
    }    

    stage('Build Project') {
        // Build the project with Maven
        sh "${mvnHome}/bin/mvn -Dmaven.test.failure.ignore clean package"
    }

    stage('Publish Tests Results') {
        parallel(
            publishJunitTestsResultsToJenkins: {
                echo "Publishing JUnit test results"
                junit '**/target/surefire-reports/TEST-*.xml'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            },
            publishJunitTestsResultsToSonar: {
                echo "This is branch B (Sonar Integration Stage - Add commands here)"
            }
        )
    }

    stage('Build Docker Image') {
        // Prepare the jar file for Docker image build
        sh "mkdir -p ./data"
        sh "mv ./target/hello*.jar ./data"
        
        // Build Docker image
        dockerImage = docker.build("${dockerImageTag}")
    }

    stage('Deploy Docker Image') {
        echo "Docker Image Tag Name: ${dockerImageTag}"

        // Push Docker image to Nexus repository
        sh "docker login -u admin -p admin123 ${dockerRepoUrl}"
        sh "docker push ${dockerImageTag}"
    }
}

