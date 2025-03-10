pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "bastienfaucher" // replace this with your docker-id
MOVIE_SERVICE_IMAGE = "movie_service"
CAST_SERVICE_IMAGE = "cast_service"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage('Debug') {
    steps {
        script {
            echo "Current branch: ${env.BRANCH_NAME}"
        }
    }
}

        stage(' Docker Build'){ 
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 docker build -t movie-service ./movie-service
                 docker build -t cast-service ./cast-service
                 docker tag movie-service:latest $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG
                 docker tag cast-service:latest $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG
                '''
                }
            }
        }
        stage('Docker Push'){ 
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") 
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG
                docker push $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG
                '''
                }
            }

        }

        stage('Deploiement en dev'){
                environment
                {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                NODEPORT = "30001"
                }
                    steps {
                        script {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp ./charts/values.yaml values.yml
                        sed -i 's/tag: latest/tag: "'"${DOCKER_TAG}"'"/g' values.yml
                        sed -i 's/nodePort: .*/nodePort: "'"${NODEPORT}"'"/g' values.yml
                        cat values.yml
                        kubectl get namespace dev || kubectl create namespace dev
                        helm upgrade --install exam-jenkins ./charts --values=values.yml --namespace dev --recreate-pods
                        '''
                        }
                    }

                }

        stage('Deploiement en QA'){
                environment
                {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                NODEPORT = "30002"
                }
                    steps {
                        script {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp ./charts/values.yaml values.yml
                        sed -i 's/tag: latest/tag: "'"${DOCKER_TAG}"'"/g' values.yml
                        sed -i 's/nodePort: .*/nodePort: "'"${NODEPORT}"'"/g' values.yml
                        cat values.yml
                        kubectl get namespace qa || kubectl create namespace qa
                        helm upgrade --install exam-jenkins ./charts --values=values.yml --namespace qa --recreate-pods
                        '''
                        }
                    }

                }

        stage('Deploiement en staging'){
                environment
                {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                NODEPORT = "30003"
                }
                    steps {
                        script {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp ./charts/values.yaml values.yml
                        sed -i 's/tag: latest/tag: "'"${DOCKER_TAG}"'"/g' values.yml
                        sed -i 's/nodePort: .*/nodePort: "'"${NODEPORT}"'"/g' values.yml
                        cat values.yml
                        kubectl get namespace staging || kubectl create namespace staging
                        helm upgrade --install exam-jenkins ./charts --values=values.yml --namespace staging --recreate-pods
                        '''
                        }
                    }

                }

        stage('Deploiement en prod manuel'){
                when { 
                    expression { env.BRANCH_NAME == 'master' }
                }
                environment
                {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                NODEPORT = "30004"
                }
                    steps {
                        timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                        }

                        script {
                        sh '''
                        rm -Rf .kube
                        mkdir .kube
                        ls
                        cat $KUBECONFIG > .kube/config
                        cp ./charts/values.yaml values.yml
                        sed -i 's/tag: latest/tag: "'"${DOCKER_TAG}"'"/g' values.yml
                        sed -i 's/nodePort: .*/nodePort: "'"${NODEPORT}"'"/g' values.yml
                        cat values.yml
                        kubectl get namespace prod || kubectl create namespace prod
                        helm upgrade --install exam-jenkins ./charts --values=values.yml --namespace prod --recreate-pods
                        '''
                        }
                    }

}
}
post { // send email when the job has failed
        // ..
        failure {
            echo "This will run if the job failed"
            mail to: "bastien.faucher.pro@gmail.com",
                subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
        // ..
    }
}