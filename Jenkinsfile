node {

    def mavenHome
    def mavenCMD
    def tagName = "3.0"

    stage('Prepare Environment') {
        echo 'Initializing Maven tool'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
    }

    stage('Git Code Checkout') {
        try {
            echo 'Checking out code from GitHub...'
            git 'https://github.com/Azharhashmi111/Insure-Me-Demo.git'
        } catch (Exception e) {
            echo 'Exception occurred during Git checkout.'
            currentBuild.result = "FAILURE"
            emailext(
                body: """Dear All,
The Jenkins job ${JOB_NAME} has failed. Please check: 
${BUILD_URL}""",
                subject: "Job ${JOB_NAME} #${BUILD_NUMBER} Failed",
                to: 'shubham@gmail.com'
            )
            error "Stopping pipeline due to checkout failure"
        }
    }

    stage('Build the Application') {
        echo "Running Maven clean package..."
        sh "${mavenCMD} clean package"
    }

    stage('Publish Test Reports') {
        echo "Publishing Surefire HTML Reports..."
        publishHTML([
            allowMissing: true,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'target/surefire-reports',
            reportFiles: 'index.html',
            reportName: 'HTML Report'
        ])
    }

    stage('Containerize the Application') {
        echo 'Creating Docker image...'
        sh "docker build -t shubhamkushwah123/insure-me:${tagName} ."
    }

    stage('Push to DockerHub') {
        echo 'Pushing Docker image to DockerHub...'
        withCredentials([string(credentialsId: 'dock-password', variable: 'dockerHubPassword')]) {
            sh "echo ${dockerHubPassword} | docker login -u shubhamkushwah123 --password-stdin"
            sh "docker push shubhamkushwah123/insure-me:${tagName}"
        }
    }

    stage('Deploy to Test Server using Ansible') {
        echo "Deploying via Ansible playbook..."
        ansiblePlaybook(
            become: true,
            credentialsId: 'ansible-key',
            disableHostKeyChecking: true,
            installation: 'ansible',
            inventory: '/etc/ansible/hosts',
            playbook: 'ansible-playbook.yml'
        )
    }
}
