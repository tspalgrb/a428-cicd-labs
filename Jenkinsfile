node {
    def awsInstanceIP = env.AWS_EC2_IP
    def appName = env.APP_NAME_REACT
    def dockerImagePush = "${env.DOCKER_HUB_USERNAME}/${appName}:latest"
    def dockerImage = docker.image('node:16-buster-slim')

    dockerImage.pull()
    dockerImage.inside('-p 3000:3000') {

        stage('Checkout Code') {
            echo 'Checkout scm...'
            checkout scm
        }
        stage('Build') {
            echo 'Installing dependencies...'
            sh 'npm install'
        }

        stage('Test') {
            echo 'Running tests...'
            sh './jenkins/scripts/test.sh'
        }

        stage('Manual Approval'){
            input message: 'Lanjutkan ke tahap Deploy? (Klik "Proceed" untuk melanjutkan ke tahap Deploy)'
        }
        
        stage('Deploy') {
            echo 'Deploying application...'
            sh './jenkins/scripts/deliver.sh'
            echo 'Waiting for 60 seconds...'
            sleep(60)
            echo 'Stopping application...'
            sh './jenkins/scripts/kill.sh'
        }
    }

    stage('Deploy EC2') {
        echo 'Deploying application...'
        echo 'Build & push docker image...' 
        sh "docker build -t ${dockerImagePush} ."
        withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
            sh """
            echo ${env.DOCKER_PASSWORD} | docker login -u ${env.DOCKER_USER} --password-stdin
            docker push ${dockerImagePush}
            """
        }
        sshagent(['ec2-ssh-key']) {
            sh """
                ssh -o StrictHostKeyChecking=no -i ${env.SSH_KEY_PATH} ${env.SSH_USER}@${awsInstanceIP} \"
                docker pull ${dockerImagePush}
                docker stop ${appName} || true
                docker rm -f ${appName} || true
                docker run -d --name ${appName} -p 3000:80 ${dockerImagePush}
                sleep 5
                docker logs ${appName}
                \"
            """
        }
        sh "docker logout"
        sleep(60)
    }
}
