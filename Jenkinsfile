try {
    node {
        def app
        stage('Clone Repository') {
            final scmVars = checkout(scm)
            env.BRANCH_NAME = scmVars.GIT_BRANCH
            env.SHORT_COMMIT = "${scmVars.GIT_COMMIT[0..7]}"
            env.GIT_REPO_NAME = scmVars.GIT_URL.replaceFirst(/^.*\/([^\/]+?).git$/, '$1')
        }
        stage

        stage('Install dependencies') {
            sh 'npm install'
        }

        stage('Lint Test') {
            sh 'npm run lint'
        }

        stage('Run Unit Tests') {
            sh 'npm run test'
        }

        stage('SonarQube code analysis') {
          withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
              sh 'sonar-scanner -Dsonar.projectKey=devsecops-kahawa-frontend -Dsonar.organization=kahawa'
          } 
        }

        stage('Build Docker Image') {
           sh "docker build --network=host -t devsecops-kahawa-frontend ."
        }

        stage('Trivy Image scan') {
          sh "trivy image -f json -o trivy-image-results.json devsecops-kahawa-frontend:latest"
        }

        stage('Push Image to Registry') {
                docker.withRegistry('https://937762161455.dkr.ecr.eu-west-1.amazonaws.com/', 'ecr:eu-west-1:awsecr-uat') {
                       sh "docker tag devsecops-kahawa-frontend 937762161455.dkr.ecr.eu-west-1.amazonaws.com/devsecops-kahawa-frontend:uat-${env.SHORT_COMMIT}"
                       sh "docker push 937762161455.dkr.ecr.eu-west-1.amazonaws.com/devsecops-kahawa-frontend:uat-${env.SHORT_COMMIT}"
                }
        }
  }
} catch (Error | Exception e) {
    //Finish failing the build after telling someone about it
    throw e
} finally {
  // Post build steps here
  /* Success or failure, always run post build steps */
// send email
// publish test results etc etc
}
def version() {
    pom = readMavenPom file: 'pom.xml'
    return pom.version
}
