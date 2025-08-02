node {
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName

    stage('Prepare Environment') {
        echo 'Initialize all the variables'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName = "3.0"
    }

    stage('Git Code Checkout') {
        try {
            echo 'Checkout the code from Git repository'
            git 'https://github.com/shubhamkushwah123/star-agile-insurance-project.git'
        } catch (Exception e) {
            echo 'Exception occurred in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
The Jenkins job ${JOB_NAME} has been failed. Please check it immediately using the below link:
${BUILD_URL}''', subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} is failed', to: 'shubham@gmail.com'
        }
    }

    stage('Build the Application') {
        echo "Cleaning... Compiling... Testing... Packaging..."
        sh "${mavenCMD} clean package"
    }

    stage('Publish Test Reports') {
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: false,
            reportDir: 'target/surefire-reports',
            reportFiles: 'index.html',
            reportName: 'HTML Report'
        ])
    }

    stage('Containerize the Application') {
        echo 'Creating Docker image'
        sh "${dockerCMD} build -t swarup98/insureme-app:${tagName} ."
    }

    stage('Push to DockerHub') {
        echo '📦 Pushing the docker image to DockerHub'
        withCredentials([string(credentialsId: 'dock-password', variable: 'dockerHubPassword')]) {
            sh "echo ${dockerHubPassword} | ${dockerCMD} login -u swarup98 --password-stdin"
            sh "${dockerCMD} push swarup98/insureme-app:${tagName}"
        }
    }

   stage('Configure and Deploy to the test-server') {
    ansiblePlaybook(
        become: true,
        credentialsId: 'ansible-key', // Jenkins Credentials → SSH Private Key
        disableHostKeyChecking: true,
        installation: 'ansible',
        inventory: 'inventory.ini',         // 👈 Your custom inventory file
        playbook: 'ansible-playbook.yml'
    )
}
