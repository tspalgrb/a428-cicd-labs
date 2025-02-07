node {
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
}
