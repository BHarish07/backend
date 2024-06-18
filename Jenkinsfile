pipeline{
    agent{
        label 'AGENT-1'
    }
    options{
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    environment{
      def  appVersion = ''
      nexusUrl = 'nexus.harishbalike.online:8081'

    }
    parameters{
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }
    stages{

        stage('read the version'){
            steps{
                script{
                    def packageJson =  readJSON file: 'package.json' 
                    appVersion = packageJson.version
                    echo 'Application version: ${appVersion}'
                }

            }
        }

        stage('install dependencies'){
            steps{
               sh """
                    sudo npm install 
                    ls -ltr 

                 """
            }
        }
        stage('Build'){
            steps{
                sh """
                zip -q -r backend-${appVersion}.zip * -x Jenkinsfile -x backend-${appVersion}.zip
                ls -ltr

                """
            }
        }

        // sonarQube 

        stage('Sonar Scan'){
            environment {
                scannerHome = tool 'sonar-6.0' //referring scanner CLI
            }
            steps {
                script {
                    withSonarQubeEnv('sonar-6.0') { //referring sonar server
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
              timeout(time: 30, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage('Nexus Artifact Upload'){
            steps{
                script{
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: "${nexusUrl}",
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: "backend",
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: "backend" ,
                            classifier: '',
                            file: "backend-" + "${appVersion}" + '.zip',
                            type: 'zip']
                        ]
                    )
                }
            }
        }
        stage('Deploy'){
            when{
                expression{
                    params.deploy
                }
            }
            steps{
                script{
                    def params = [
                        string(name: 'appVersion', value: "${appVersion}")
                    ]
                    build job: 'backend-deploy', parameters: params, wait: false
                }
            }
        }

    }
     post { 
        success {
            echo 'The pipeline executed successfully'
        }
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
         failure { 
            echo 'I will run when pipeline is failure'
        }
     }

}