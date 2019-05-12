pipeline {
    agent { label "master"}
    stages {
        stage('Delete the workspace') {
            steps {
                sh "sudo rm -rf $WORKSPACE/*"
            }
        }

        stage('Installing ChefDK') {
            steps {
                script {
                    def exists = fileExists '/usr/bin/chef-client'
                    if (exists) {
                        echo "Skipping ChefDK install - already installed!"
                    } else {
                        sh 'sudo apt-get install -y wget'
                        sh 'wget https://packages.chef.io/files/stable/chefdk/3.8.14/ubuntu/18.04/chefdk_3.8.14-1_amd64.deb'
                        sh 'sudo dpkg -i chefdk_3.8.14-1_amd64.deb'
                    }
                }

            }
        }

        stage('Installing Habitat') {
            steps {
                script {
                    def exists = false //need to test if the hab cli is working (>hab --version returns a good value)
                    if (exists) {
                        echo "Skipping Habitat install - already installed!"
                    } else {
                        sh 'sudo apt-get install -y curl'
                        sh 'curl https://raw.githubusercontent.com/habitat-sh/habitat/master/components/hab/install.sh | sudo bash'
                        sh 'useradd hab'
                        sh 'usermod -aG hab hab'
                    }
                }

            }
        }

        stage('Loading the Hab Package') {
            steps {
                script{
                    sh 'nohup sudo hab sup run $>/dev/null &'
                    sh 'sudo hab svc load jnalewak-hab-pub/mongodb-parks'
                    sh 'sudo hab svc load jnalewak-hab-pub/national-parks --bind database:mongodb-parks.default'
                }
            }
        }

        stage('Send Slack Notification') {
            steps{
                slackSend color: '#449FE0', message: "Jason Nalewak: Please approve ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.JOB_URL}|Open>)"
            }
            
        }

        stage('Wait for Input'){
            steps{
                input 'Proceed with build?'
            }
        }

        stage('Push to Stable Channel in Builder') {
            steps {
                //promote package to stable channel
            }
        }

        stage('')
    }
    //post actions
    post {
        success {
        slackSend color: '#439FE0', message: "Build $JOB_NAME $BUILD_NUMBER was successful"
        }
        failure {
            echo "Build Failed"
            mail  body: "Build ${env.JOB_NAME} ${env.BUILD_NUMBER} failed. Please check the build at ${env.JOB_URL}", from: 'admin@acme.com', subject: 'Build Failure', to: 'jnalewak@chef.io'
        }
    }

}