node {
    def mavenHome
    def mavenCMD
    def tagName = "3.0"
    def dockerImage = "azharhashmi/insure-me:${tagName}"

    stage('Prepare Environment') {
        echo 'Initializing Maven tool...'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
    }

    stage('Git Code Checkout') {
        try {
            echo 'Cloning code from GitHub...'
            git 'https://github.com/Azharhashmi111/Insure-Me-Demo.git'
        } catch (Exception e) {
            echo 'Exception occurred during Git checkout.'
            currentBuild.result = "FAILURE"
            emailext(
                body: """Dear All,
The Jenkins job ${JOB_NAME} has failed during Git checkout. Please review the details here:
${BUILD_URL}""",
                subject: "Job ${JOB_NAME} #${BUILD_NUMBER} Failed",
                to: 'azhar@gmail.com'
            )
            error "Stopping pipeline due to Git checkout failure."
        }
    }

    stage('Generate Inventory File') {
        echo 'Creating dynamic inventory.ini file...'
        writeFile file: 'inventory.ini', text: '''
[test]
100.26.193.218 ansible_user=ubuntu
'''
    }

    stage('Build the Application') {
        echo 'Running Maven clean package...'
        sh "${mavenCMD} clean package"
    }

    stage('Publish Test Reports') {
        echo 'Publishing Surefire HTML reports...'
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
        echo "Building Docker image: ${dockerImage}"
        sh "docker build -t ${dockerImage} ."
    }

    stage('Push to DockerHub') {
        echo "Pushing Docker image to DockerHub as ${dockerImage}"
        withCredentials([string(credentialsId: 'dock-password', variable: 'DOCKER_PASSWORD')]) {
            sh '''
                echo "$DOCKER_PASSWORD" | docker login -u azharhashmi --password-stdin
                docker push ''' + dockerImage
        }
    }

    stage('Deploy to Test Server using Ansible') {
        echo "Deploying application using Ansible..."

        withCredentials([sshUserPrivateKey(credentialsId: 'ansible-key', keyFileVariable: 'ANSIBLE_KEY')]) {
            sh '''
                ansible-playbook ansible-playbook.yml \
                    -i inventory.ini \
                    --private-key $ANSIBLE_KEY \
                    -u ubuntu \
                    -b
            '''
        }
    }
}
