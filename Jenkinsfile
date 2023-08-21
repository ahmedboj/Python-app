pipeline{
    agent{
    	docker {
            image 'ubuntu:latest' 
            args '-u root:sudo -v $HOME/workspace/myproject:/myproject'
        }
    }
    parameters{
        string(name: 'Host', defaultValue: '192.168.102.82', description: 'The host Ip address for K8s master node')
    }

    environment{
        DOCKER_IMAGE = "192.168.102.81:5000/python-app"
        DOCKER_TAG = "1.0"
    }

    stages{
        
        
        stage("Test the application"){
            steps{
                echo "Running application tests"
                sh 'apt-get update && apt-get install make git python3.9 python3-venv python3-dev gcc -y'
                sh 'apt-get update && apt-get install make git python3.9 python3-venv python3-dev gcc -y'
                sh 'update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9'
                sh 'python3 -m venv src/.venv'
                sh 'make test'
            }

        }

        stage("Build Docker Image"){
            steps{
                echo "Building Docker Image"
                withCredentials([usernamePassword(credentialsId:'NEXUS_LOGIN', passwordVariable: 'PWD', usernameVariable: 'USER')]){
                    sh "echo Username: \${USER}"
                    sh "echo ${PWD} | docker login -u ${USER} --password-stdin 192.168.102.81:5000"
                    sh 'docker build -t python-app:1.0 . '
                    sh 'docker tag python-app:1.0 ${DOCKER_IMAGE}:${DOCKER_TAG} '
                    sh 'docker push ${DOCKER_IMAGE}:${DOCKER_TAG} '

                }

            }
        }
        stage("Deploy python application in K8S"){
            steps{
                echo "SSH into K8s master node"
                script{
                        def remote = [:]
                        remote.name = "master-pfe-3"
                        remote.host = params.Host
                        remote.allowAnyHosts = true

                        withCredentials([sshUserPrivateKey(credentialsId: 'ID_K8S', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {

                                remote.user = userName
                                remote.identityFile = identity
                                sshCommand remote: remote, command: "echo Hello from remote host"
                                sshCommand remote: remote, command: "docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}"
                                sshCommand remote: remote, command: "docker run -d -p 8000:8000 ${DOCKER_IMAGE}:${DOCKER_TAG}"

                        }
                }

            }
        }

    }
    post{
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}

