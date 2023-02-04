pipeline {
    agent { label 'agent1' }
    stages {
        stage('build') {
            steps {
                script {
                   if (env.BRANCH_NAME == "release") {
                       withCredentials([usernamePassword(credentialsId: 'dockerhub_cred', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                           sh """
                                docker login -u $USERNAME -p $PASSWORD
                                docker build -t aomarabdelaziz/itimansbakehouse:${BUILD_NUMBER} .
                                docker push aomarabdelaziz/itimansbakehouse:${BUILD_NUMBER}
                                echo ${BUILD_NUMBER} > ../bakehouse-build-number.txt
                           """
                       }
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == "dev" || env.BRANCH_NAME == "test" || env.BRANCH_NAME == "prod") {
                            withCredentials([file(credentialsId: 'jenkins-secretFile', variable: 'KUBECONFIG')]) {
                          sh """
                              export BUILD_NUMBER=\$(cat ../bakehouse-build-number.txt)
                              mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                              cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                              rm -f Deployment/deploy.yaml.tmp
                              kubectl apply -f Deployment --kubeconfig=${KUBECONFIG}
                            """
                        }
                    }
                }
            }
        }
    }
}
