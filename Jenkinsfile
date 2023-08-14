pipeline{
    agent any
    
    parameters{
        string(name: 'Host', defaultValue: '', description: 'The host Ip address for K8s master node')
    }
    
    
    stages{
        stage("Build the Application"){
            steps{
                echo "Building Jar file"
                sh './gradlew build'
            }

        }

        stage("Build the Docker Image"){
            steps{
                echo "Building Docker Image"
                withCredentials([usernamePassword(credentialsId:'NEXUS_LOGIN', passwordVariable: 'PWD', usernameVariable: 'USER')]){
                    sh "echo Username: \${USER}"
                    sh "echo ${PWD} | docker login -u ${USER} --password-stdin 192.168.102.81:5000"
                    sh 'docker build -t demo-app:v1.0 . '
                    sh 'docker tag demo-app:v1.0 192.168.102.81:5000/demo-app:v1.0 '
                    sh 'docker push 192.168.102.81:5000/demo-app:v1.0 '

                }

            }
        }
        stage("SSH into K8S"){
            steps{
                echo "SSH into K8s master node"
		def remote = [:]
		remote.host = params.Host
		remote.allowAnyHosts = true
		withCredentials([sshUserPrivateKey(credentialsId: 'ID_K8S', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
  		    remote.user = userName
      		    remote.identityFile = identity

        	    sshCommand remote: remote, command: "echo Hello from remote host"
        	    sshCommand remote: remote, command: "docker pull 192.168.102.81:5000/demo-app:v1.0"
                    sshCommand remote: remote, command: "kubectl apply -f /home/cloud-user/springboot-with-docker/k8s-spring-boot-deployment.yml"
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
