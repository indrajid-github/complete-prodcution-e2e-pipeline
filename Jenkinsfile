pipeline
{
    agent
    {
        node
        {
            label 'maven'
        }
    }
    tools
    {
        maven 'maven3'
        jdk 'jdk17'
    }
    environment
    {
        // For docker cred
        DOCKER_USER = "uriyapraba"
        DOCKER_PASS = "dockerhub"
        //App name and release
        APP_NAME = "complete-prodcution-e2e-pipeline"
        RELEASE = "1.0.0"
        //For Docker image and tag
        IMAGE_NAME = "${DOCKER_USER }" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials('JENKINS_API_TOKEN')
    }
    stages
    {
        stage("Cleanup workspace")
        {
            steps
            {
                cleanWs()
            }
        }
        stage("Checkout SCM")
        {
            steps
            {
                script
                {
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github_tocken', url: 'https://github.com/indrajid-github/complete-prodcution-e2e-pipeline.git']])
                }
                
            }
        }
        stage("Build Maven Project")
        {
            steps
            {
                sh 'mvn clean package'
            }
        }
        stage("Build Maven Test")
        {
            steps
            {
                sh 'mvn test'
            }
        }
        stage("SonarQube Analysis")
        {
            steps
            {
                script
                {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonar-token') 
                    {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage("Quality Gate") // sonarqube webhook is neccesory to create to send the response 
        {
            options
            {
                timeout(time: 100, unit: 'SECONDS') //If the step is not satisfied within 100 sec's, then the build will be aborted
            }
            steps
            {
                script
                {
                    def qa = waitForQualityGate()
                    if(qa.status != 'OK')
                    {
                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    }
                }
            }
        }
        stage("Build and Push Docker Image")
        {
            steps
            {
                script
                {
                    docker.withRegistry('', DOCKER_PASS) 
                    {
                        docker_image = docker.build("${IMAGE_NAME}")      
                    }
                    docker.withRegistry('', DOCKER_PASS) 
                    {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push 'latest'     
                    }
                }
            }
            post
            {
                success
                {
                   sh "docker image rm ${IMAGE_NAME}:${IMAGE_TAG}"
                   sh "docker image rm ${IMAGE_NAME}:latest" 
                }
            }
        }
        stage("Trigger CD")
        {
            steps
            {
                script
                {
                    sh "curl -vk --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://3.106.192.148:8080/job/github-complete-pipeline/buildWithParameters?token=github'"
                }
            }
        }
    }
}