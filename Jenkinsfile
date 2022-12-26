pipeline {
    agent any
    environment {
        def git_branch = 'master'
        def git_url = 'https://github.com/avidere/Java_App.git'

        def mvntest = 'mvn test '
        def mvnpackage = 'mvn clean install'

        def sonar_cred = 'sonar'
        def code_analysis = 'mvn clean install sonar:sonar'

        def nex_cred = 'nexus'
        def grp_ID = 'com.example'
        def nex_url = '172.31.28.226:8081'
        def nex_ver = 'nexus3'
        def proto = 'http'
        def repo = 'demoproject'
        def utest_url = 'target/surefire-reports/**/*.xml'
    }
    stages{
        stage('Git Checkout'){
            steps{
                script{
                    git branch: "${git_branch}", url: "${git_url}"
                    echo "Git Checkout Completed"
                }
            }
        }
        stage('Maven Build'){
            steps{
                sh "${env.mvnpackage}"
                echo "Maven Build Completed"
            }
        }
        stage('Unit Testing and Code Coverage report'){
            steps{
                script {
                        junit "$utest_url"
                        jacoco()
                }
            }
        }
        stage('Static code analysis') {
            steps{
                script{
                    withSonarQubeEnv(credentialsId: "${sonar_cred}") {
                        sh "${code_analysis}"
                        echo"Static Code Analysis"
                    }
                }
            }
        }
        stage('Quality Gate Status') {
                steps{
                    script{
                        waitForQualityGate abortPipeline: true, credentialsId: "${sonar_cred}"
                        echo "Quality Gate status Completed"
                    }
                }
        }
        stage('Upload Artifact to nexus repository') {
            steps {
                script {
                    
                    def mavenpom = readMavenPom file: 'pom.xml'
                    def nex_repo = mavenpom.version.endsWith('SNAPSHOT') ? 'demoproject-snapshot' : 'demoproject-Release'
                    nexusArtifactUploader artifacts: [
                    [
                        artifactId: 'Java_app',
                        classifier: '',
                        file: "target/Java_app-${mavenpom.version}.war",
                        type: 'war'
                    ]
                ],
                    credentialsId: "${env.nex_cred}",
                    groupId: "${env.grp_ID}",
                    nexusUrl: "${env.nex_url}",
                    nexusVersion: "${env.nex_ver}",
                    protocol: "${env.proto}",
                    repository: "${nex_repo}",
                    version: "${mavenpom.version}"
                    echo 'Artifact uploaded to nexus repository'
                }
            }
        }
    }
}
