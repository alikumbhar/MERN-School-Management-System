@Library('Shared') _
pipeline {
    agent {label 'localmachine'}
    
    environment{
        SONAR_HOME = tool "Sonar"
    }
    
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    }
    // all stage will be in under Stages portion    
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }
        //cleaning Workspace before start
        stage("Workspace cleanup"){
            steps{
                script{
                    cleanWs()
                }
            }
        }

        // clone code from git repo
        stage('Git Repo Clone') {
            steps {
                script{
                    clone("https://github.com/alikumbhar/MERN-School-Management-System.git","main")
                }
            }
        }
        //end of code clone git repo



        //DevSecOps Stages Started
        stage("Trivy Scanner"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }

        stage("OWASP Dependency Checker"){
            steps{
                //this will check current directory for owasp installation
               // and then generate report in xml format
                script{
                    dependencyChecker_owasp()
                }
            }
        }
        
        stage("CodeAnalysis: SonarCode Analysis"){
            steps{
                script{
                    analyzeSonar("Sonar","sms-app","sms-app")
                }
            }
        }
        
        stage("SonarQubeQualityBuild"){
            //this will check sonar quality gate statusd

            steps{
                script{
                    qualityCodeBuilderSonar()
                }
            }
        }
        //DevSecOps Stages Ended

        //Docker Stages Started
        stage("Docker: Build Images and Tagging"){
            steps{
                script{
                        dir('backend'){
                            docker_build("wanderlust-backend-beta","${params.BACKEND_DOCKER_TAG}","trainwithshubham")
                        }
                    
                        dir('frontend'){
                            docker_build("wanderlust-frontend-beta","${params.FRONTEND_DOCKER_TAG}","trainwithshubham")
                        }
                }
            }
        }
        
        stage("DockerPush: Pushing Docker Images to DockerHub"){
            steps{
                script{
                    docker_push("mern-apps","${params.BACKEND_DOCKER_TAG}","alikumbhar") 
                    docker_push("mern-apps","${params.FRONTEND_DOCKER_TAG}","alikumbhar")
                }
            }
        }
        //Docker Stages Ended
    } //end of stages

    post{

        success{
            archiveArtifacts artifacts: '*.xml', followSymlinks: false
            build job: "SMS-Application-CD", parameters: [
                string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
            ]
        }
    }
    //if success system will trigger SMS-Application-CD job using build job

}