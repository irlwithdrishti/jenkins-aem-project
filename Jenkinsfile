pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'deploy',
                    url: 'git@github.com:irlwithdrishti/jenkins-aem-project.git',
                    credentialsId: 'git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Copy Package') {
            steps {
                sh 'mkdir -p /opt/aem-packages && cp all/target/*.zip /opt/aem-packages/'
            }
        }

        stage('Ensure Author Running') {
            steps {
                sh '''
                    if ! curl -s -o /dev/null http://localhost:4502; then
                        /mnt/aem/author/crx-quickstart/bin/start
                        sleep 30
                    fi
                '''
            }
        }

        stage('Ensure Publish Running') {
            steps {
                sh '''
                    if ! curl -s -o /dev/null http://localhost:4503; then
                        /mnt/aem/publish/crx-quickstart/bin/start
                        sleep 30
                    fi
                '''
            }
        }

        stage('Deploy to Author') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aem-author-creds', usernameVariable: 'AEM_USER', passwordVariable: 'AEM_PASS')]) {
                    sh '''
                        PKG=$(ls /opt/aem-packages | tail -1)
                        curl -u $AEM_USER:$AEM_PASS -F file=@/opt/aem-packages/$PKG -F name=$PKG -F force=true -F install=true \
                        http://localhost:4502/crx/packmgr/service.jsp
                    '''
                }
            }
        }

        stage('Deploy to Publish') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aem-publish-creds', usernameVariable: 'AEM_USER', passwordVariable: 'AEM_PASS')]) {
                    sh '''
                        PKG=$(ls /opt/aem-packages | tail -1)
                        curl -u $AEM_USER:$AEM_PASS -F file=@/opt/aem-packages/$PKG -F name=$PKG -F force=true -F install=true \
                        http://localhost:4503/crx/packmgr/service.jsp
                    '''
                }
            }
        }
    }
}
