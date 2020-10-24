pipeline {
    agent any
       triggers {
        pollSCM "* * * * *"
       }
    stages {
        stage('Build AWS Demo Application') { 
            steps {
                echo '=== Building Demo Application ==='
                sh 'mvn -B -DskipTests clean package' 
            }
        }
        stage('Test AWS Demo Application') {
            steps {
                echo '=== Testing Demo Application ==='
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Building Demo Docker Image ==='
                script {
                    app = docker.build("prasadadireddi/awsdemo-spinnaker-jenkins")
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Pushing Petclinic Docker Image ==='
                script {
                    GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
                    SHORT_COMMIT = "${GIT_COMMIT_HASH[0..7]}"
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerHubCredentials') {
                        app.push("$SHORT_COMMIT")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Remove local images') {
            steps {
                echo '=== Delete the local docker images ==='
                sh("docker rmi -f ibuchh/awsdemo-spinnaker-jenkins:latest || :")
                sh("docker rmi -f ibuchh/awsdemo-spinnaker-jenkins:$SHORT_COMMIT || :")
            }
        }
    }
}
