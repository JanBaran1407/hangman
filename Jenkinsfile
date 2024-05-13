pipeline {
    agent any

    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('Pull') {
            steps {
                git credentialsId: '12fadcc3-1873-4f15-8bc5-08ea4ad9e825', url: 'https://github.com/JanBaran1407/hangman.git'
            }
        }
        
        stage('Clean up') {
            steps {
                echo "Cleaning up wrokspace..."
                sh '''
                docker image rm -f build_stage
                docker image rm -f test_stage
                docker image rm -f deploy_stage
                
                docker stop artifact
                docker container rm artifact
                '''
            }
        }

        stage('Build') {
            steps {
                echo "Building..."
                sh '''
                cd dockerfiles
                docker build -f DockerfileBuild -t build_stage .
                docker run --name build_container hangman:latest
                docker cp build_container:/hangman/build ./artefakty
                docker logs build_container > log_build.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo "Testing..."
                sh '''
                cd dockerfiles
                docker build --no-cache -f DockerfileTest -t test_stage .
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                echo "Deploying..."
                sh '''
                cd dockerfiles
                docker-compose up -d
                docker build --no-cache -f dockerfileDeploy -t deploy_stage .
                docker run --rm deploy_stage
                '''
            }
        }

        stage('Finish') {
            steps {
                echo "Finishing..."
                sh '''
                cd dockerfiles
                TIMESTAMP=$(date +%Y%m%d%H%M%S)
                tar -czf artifact_$TIMESTAMP.tar.gz log_build.txt log_test.txt
                docker-compose down
                '''
            }
        }
        stage('Publish') {
            steps {
                echo "Publish stage"
                sh '''
                cd dockerfiles
                TIMESTAMP=$(date +%Y%m%d%H%M%S)
                tar -czf artifact_$TIMESTAMP.tar.gz log_build.txt log_test.txt artefakty
                
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                NUMBER='''+ env.BUILD_NUMBER +'''
                docker tag hangman-deploy:latest JanBaran1407/hangman:latest
                docker push JanBaran1407/hangman:latest
                docker logout

                '''
            } 
        }
        
    }
    post {
        always {
            echo "Archiving artifacts..."
            archiveArtifacts artifacts: 'dockerfiles/*.tar.gz', fingerprint: true
        }
    }
}
