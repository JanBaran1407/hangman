pipeline {
    agent any

    triggers {
        pollSCM('* * * * *')
    }
    environment {
        DOCKERHUB_TOKEN = credentials('dockerhub_token')
    }
    stages {
        stage('Pull') {
            steps {
                git credentialsId: '12fadcc3-1873-4f15-8bc5-08ea4ad9e825', url: 'https://github.com/JanBaran1407/hangman.git'
            }
        }


        stage('Build') {
            steps {
                echo "Building..."
                sh '''
                cd dockerfiles
                docker build -f DockerfileBuild -t build_stage .
                docker run --name build_container8 build_stage
                docker cp build_container8:/app/build ./artefakty
                docker logs build_container8 > log_build.txt
                docker rm build_container8
                '''
            }
        }

        stage('Test') {
            steps {
                echo "Testing..."
                sh '''
                cd dockerfiles
                docker build -f DockerfileBuild -t build_stage .
                docker build -t test_stage -f DockerfileTest  .
                docker run --name test_container test_stage 
                docker logs test_container > log_build.txt
                docker rm test_container
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                echo "Deploying..."
                echo "Saving Working gallows.py file as artefact"

            }
        }

        stage('Publish') {
            steps {
                echo "Publish stage"
                sh '''
                cd dockerfiles
                
                echo $DOCKERHUB_TOKEN | docker login -u janbaran1407 -p $DOCKERHUB_TOKEN
                docker build -f DockerfileBuild -t janbaran1407/hangman:latest .
                docker push janbaran1407/hangman:latest
                docker logout

                '''
            } 
        }
        
    }
    post {
        always {
            archiveArtifacts artifacts: 'gallows.py, artefakty', onlyIfSuccessful: true
        }
    }
}
