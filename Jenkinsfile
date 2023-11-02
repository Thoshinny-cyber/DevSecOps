pipeline{
    agent any
    tools {
      maven 'maven'
    }
    environment {
      DOCKER_TAG = getVersion()
      //SNYK_API = env.SNYK_API_TOKEN
    }
    stages{
        stage('SCM'){
            steps{
                git credentialsId: 'github', 
                    url: 'https://github.com/Thoshinny-cyber/DevSecOps.git'
            }
        }
        stage ('Check-Git-Secrets') {
      steps {
        sh "rm trufflehog || true"
        sh "docker pull gesellix/trufflehog"
        sh "docker run gesellix/trufflehog --json https://github.com/Thoshinny-cyber/DevSecOps.git > trufflehog"
        sh "cat trufflehog"
      }
    }
         
         stage ('Source Composition Analysis') {
      steps {
         sh 'rm owasp* || true'
         sh 'wget "https://raw.githubusercontent.com/Thoshinny-cyber/DevSecOps/master/owasp-dependency-check.sh" '
         sh 'chmod +x owasp-dependency-check.sh'
         sh 'bash owasp-dependency-check.sh'
         sh 'cat /var/lib/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
        
      }
    }

      stage ('SAST') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
          sh 'cat target/sonar/report-task.txt'
        }
      }
    }//   stage('SAST') {
      // steps {
        // timeout(time: 10, unit: 'MINUTES') {
        // echo 'Testing...'
        // snykSecurity(
        //   failOnError: false, 
        //   failOnIssues: false, 
        //   snykInstallation: 'Snyk', 
        //   snykTokenId: 'Snyk'
        //     )
          // place other parameters here
        // )
        // }
          // script{
          // // withCredentials([string(credentialsId: 'env.SNYK_API_TOKEN', variable: 'SNYK_API_TOKEN')]) 
          //               sh "docker pull thoshinny/snyk:latest"
          //               sh "docker run thoshinny/"
          //               // Run the Snyk scan within the Docker container
          //               sh "snyk auth ${env.SNYK_API_TOKEN}"
          //               sh "snyk test"
                    
          //  }
          // }
     // }
   // }        
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build'){
            steps{
                sh "docker build . -t thoshinny/devsecops:${DOCKER_TAG} "
            }
        }
        
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker_hub1', variable: 'dockerHubPwd')]) {
                    sh "docker login -u thoshinny -p ${dockerHubPwd}"
                }
                
                sh "docker push thoshinny/devsecops:${DOCKER_TAG} "
            }
        }
        
        stage('Docker Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        }
        stage ('DAST') {
        steps {
        sshagent(['zap']) {
         sh 'ssh -o  StrictHostKeyChecking=no ec2-user@3.109.123.52 "docker run -t owasp/zap2docker-stable zap-baseline.py -t http://13.233.96.201:8080/dockeransible/" || true'
        }
      }
    }
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
