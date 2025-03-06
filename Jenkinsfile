pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "bastienfaucher" // replace this with your docker-id
MOVIE_SERVICE_IMAGE = "movie_service"
CAST_SERVICE_IMAGE = "cast_service"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 docker-compose down
                 docker-compose build
                 docker tag examjenkins_movie_service:latest $DOCKER_ID/$MOVIE_SERVICE_IMAGE:$DOCKER_TAG
                 docker tag examjenkins_cast_service:latest $DOCKER_ID/$CAST_SERVICE_IMAGE:$DOCKER_TAG
                sleep 6
                '''
                }
            }
        }
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker-compose up -d
                    sleep 20
                    '''
                    }
                }
            }

        stage('Test'){ // we launch the curl command to validate that the container responds to the request
            steps {
        script {
            def cast_response = sh(script: '''
                curl -X 'POST' \
                    'http://localhost:8081/api/v1/casts/' \
                    -H 'accept: application/json' \
                    -H 'Content-Type: application/json' \
                    -d '{
                    "name": "Leonardo",
                    "nationality": "Amerloc"
                    }' \
                    -w "%{http_code}" -o /dev/null
            ''', returnStdout: true).trim()

            def movie_response = sh(script: '''
                curl -X 'POST' \
                    'http://localhost:8081/api/v1/movies/' \
                    -H 'accept: application/json' \
                    -H 'Content-Type: application/json' \
                    -d '{
                    "name": "movie test",
                    "plot": "c'\''est un test d'\''api",
                    "genres": [
                        "test"
                    ],
                    "casts_id": [
                        1
                    ]
                    }' \
                    -w "%{http_code}" -o /dev/null
            ''', returnStdout: true).trim()

            
            if (cast_response == '201' && movie_response == '201') {
                echo "Test successful: Both cast and movie created successfully with status 201."
            } else {
                error "Test failed: One or both requests did not return status 201. Cast response: ${cast_response}, Movie response: ${movie_response}"
            }
        }
    }

        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
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

//         stage('Deploiement en dev'){
//                 environment
//                 {
//                 KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
//                 }
//                     steps {
//                         script {
//                         sh '''
//                         rm -Rf .kube
//                         mkdir .kube
//                         ls
//                         cat $KUBECONFIG > .kube/config
//                         cat $KUBECONFIG
//                         cp fastapi/values.yaml values.yml
//                         cat values.yml
//                         sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
//                         helm upgrade --install app fastapi --values=values.yml --namespace dev
//                         '''
//                         }
//                     }

//                 }
//         stage('Deploiement en staging'){
//                 environment
//                 {
//                 KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
//                 }
//                     steps {
//                         script {
//                         sh '''
//                         rm -Rf .kube
//                         mkdir .kube
//                         ls
//                         cat $KUBECONFIG > .kube/config
//                         cp fastapi/values.yaml values.yml
//                         cat values.yml
//                         sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
//                         helm upgrade --install app fastapi --values=values.yml --namespace staging
//                         '''
//                         }
//                     }

//                 }
//         stage('Deploiement en prod'){
//                 environment
//                 {
//                 KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
//                 }
//                     steps {
//                     // Create an Approval Button with a timeout of 15minutes.
//                     // this require a manuel validation in order to deploy on production environment
//                             timeout(time: 15, unit: "MINUTES") {
//                                 input message: 'Do you want to deploy in production ?', ok: 'Yes'
//                             }

//                         script {
//                         sh '''
//                         rm -Rf .kube
//                         mkdir .kube
//                         ls
//                         cat $KUBECONFIG > .kube/config
//                         cp fastapi/values.yaml values.yml
//                         cat values.yml
//                         sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
//                         helm upgrade --install app fastapi --values=values.yml --namespace prod
//                         '''
//                         }
//                     }

//                 }


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