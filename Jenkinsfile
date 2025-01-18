node {
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName

    stage('prepare environment') {
        echo 'initialize all the variables'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName = "3.0"
    }

    stage('git code checkout') {
        try {
            echo 'checkout the code from git repository'
            git 'https://github.com/shubhamkushwah123/star-agile-insurance-project.git'
        } catch (Exception e) {
            echo 'Exception occurred in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
            The Jenkins job ${JOB_NAME} has failed. Please look into it immediately by clicking on the link below:
            ${BUILD_URL}''', subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} Failed', to: 'shubham@gmail.com'
        }
    }

    stage('Build the Application') {
        echo "Cleaning... Compiling... Testing... Packaging..."
        sh "${mavenCMD} clean package"
    }

    stage('Publish test reports') {
        // Update path to a relative or variable path if necessary
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, 
            reportDir: 'target/surefire-reports', 
            reportFiles: 'index.html', 
            reportName: 'HTML Report', 
            reportTitles: '', 
            useWrapperFileDirectly: true])
    }

    stage('Containerize the application') {
        echo 'Creating Docker image'
        sh "${dockerCMD} build -t shubhamkushwah123/insure-me:${tagName} ."
    }

    stage('Push to DockerHub') {
        echo 'Pushing the Docker image to DockerHub'
        withCredentials([string(credentialsId: 'dock-password', variable: 'dockerHubPassword')]) {
            sh "${dockerCMD} login -u shubhamkushwah123 -p ${dockerHubPassword}"
            sh "${dockerCMD} push shubhamkushwah123/insure-me:${tagName}"
        }
    }

    stage('Configure and Deploy to the test-server') {
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
