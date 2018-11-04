def label = "my-pod-${UUID.randomUUID().toString()}"
podTemplate(label: label,
        containers: [
                containerTemplate(name: 'build-agent',
                        image: '${DOCKER_REPOSITORY}/tests:1.0',
                        ttyEnabled: true,
                        command: 'cat',
                        privileged: true,
                        resourceRequestCpu: '2',
                        resourceRequestMemory: '4Gi'
                )
        ],
        volumes: [
                hostPathVolume(
                        mountPath: '/var/run/docker.sock',
                        hostPath: '/var/run/docker.sock')

        ])
        {
            timestamps {

                node(label) {
                    try {
                        container('build-agent') {

                            stage('git clone') {
                                echo 'git checkout'
                                checkout scm
                            }

                            stage('Installing Dependencies') {
                                echo 'Installing Dependencies'
                                sh 'npm i express --save
                                sh 'npm i mocha --save-dev'
                            }

                            stage('Unit Test') {
                                echo 'Running tests'
                            }

                            stage('Package') {
                                echo 'Packaging'
                                sh 'docker build bigid:${IG_IMAGE_TAG} .'
                            }

                            stage('Deploy to Env (CI)') {
                                echo 'Deploying to CI'
                            }

                             stage('System Test') {
                                echo 'Running system tests'
                                sh 'npm test'
                                //todo: fail build on tests
                             }

                            stage('Publish to docker') {
                                     withCredentials([[$class: 'UsernamePasswordMultiBinding',
                                         credentialsId: 'ADD JENKINS CREDENTIALS',
                                         usernameVariable: 'DOCKER_HUB_USER',
                                         passwordVariable: 'DOCKER_HUB_PASSWORD']]) {
                                             sh "docker login -u $DOCKER_HUB_USER -p $DOCKER_HUB_PASSWORD ${DOCKER_REPOSITORY}; docker tag ${DOCKER_REPOSITORY}/bigid:${IG_IMAGE_TAG} ;docker push ${DOCKER_REPOSITORY}/bigid:${IG_IMAGE_TAG}"
                                         }
                            }

                            stage('Push tag to repository'){
                                echo 'New version is..'
                            }
                        }
                    }
                    catch (exc) {
                        println "Build failed - ${currentBuild.fullDisplayName}"
                        //todo: notifyByEmail
                        throw (exc)
                    } finally {
                    }
                }
            }
        }


