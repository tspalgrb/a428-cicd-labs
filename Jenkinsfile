node {
    def dockerImage = docker.image('node:16-buster-slim')

    dockerImage.pull()
    dockerImage.inside('-p 3000:3000') {
        try {
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

            stage('Deploy') {
                echo 'Deploying application...'
                sh './jenkins/scripts/deliver.sh'
                input message: 'Sudah selesai menggunakan React App? (Klik "Proceed" untuk mengakhiri)'
                echo 'Stopping application...'
                sh './jenkins/scripts/kill.sh'
            }
        } finally {
            echo 'Pipeline finished.'
        }
    }
}
