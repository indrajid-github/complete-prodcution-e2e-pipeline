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
        stage("Quality Gate")
        {
            options
            {
                timeout(time: 100, unit: 'SECONDS')
            }
            steps
            {
                def qa = waitForQualityGate()
                if(qa.status =! 'OK')
                {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
            }
        }
    }
}