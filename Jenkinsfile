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
    }
}