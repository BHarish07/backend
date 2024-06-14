pipeline{
    agent{
        label 'AGENT-1 '
    }
    options{
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentbuilds()
        ansiColor('xterm')
    }
    environment{
      def  appVersion = ''

    }
    stages{
        stage('read the version'){
            steps{
                script{
                    def packageJson =  readJSON file: 'package.json' 
                    appVersion = packageJson.version
                    echo 'Application version: $appVersion'
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

    }
     post { 
        success {
            echo ' The pipeline executed successfully'
        }
        always { 
            echo 'I will always say Hello again!'
            deleteDir()
        }
}