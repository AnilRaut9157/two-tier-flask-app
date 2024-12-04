pipeline {
    agent any

    environment {
        SONAR_HOME = tool "sonar"
    }
    
    stages {
        stage("Checkout Code") {
            steps {
                git url: "https://github.com/AnilRaut9157/two-tier-flask-app.git", branch: "master"
            }
        }
        
        stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("sonar") {
                    withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                        sh '''
                        $SONAR_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=flaskapp \
                        -Dsonar.projectName=flaskapp \
                        -Dsonar.host.url=http://52.156.130.1:9000/ \
                        -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }
        
        stage("Build Docker Image") {
            steps {
                sh "docker build . -t flaskapp"
            }
        }
        
        stage("Push to DockerHub") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPass', usernameVariable: 'dockerHubUser')]) {
                    sh '''
                    docker login -u ${dockerHubUser} -p ${dockerHubPass}
                    docker tag flaskapp ${dockerHubUser}/flaskapp:latest
                    docker push ${dockerHubUser}/flaskapp:latest
                    '''
                }
            }
        }
        
        stage("Deploy Application") {
            steps {
                sh '''
                docker-compose down
                docker-compose up -d
                '''
            }
        }
    }
    
    post {
        always {
            echo "Pipeline execution completed."
        }
        success {
            echo "Pipeline executed successfully."
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
