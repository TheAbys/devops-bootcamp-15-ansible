pipeline {   
  agent any
  environment {
    ANSIBLE_SERVER = "138.68.94.71"
  }
  stages {
    stage("copy files to ansible server") {
      steps {
        script {
          echo "copying all neccessary files to ansible control node"

          // use ansible-server-key (configured in Jenkins) to be able to ssh into the server
          sshagent(['ansible-server-key']) {
            // copy ansible files to the ansible server
            sh "scp -o StrictHostKeyChecking=no ansible/* root@${ANSIBLE_SERVER}:/root"

            // copy the ec2-server-key (configured in Jenkins) to the ansible server
            withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
              sh 'scp $keyfile root@$ANSIBLE_SERVER:/root/ssh-key.pem'
            }
          }
        }
      }
    }
    stage ("execute ansible playbook") {
      steps {
        script {
          echo "calling ansible playbook to configure ec2 instances"
          // configure were to connect to
          def remote = [:]
          remote.name = "ansible-server"
          remote.host = ANSIBLE_SERVER
          remote.allowAnyHosts = true

          // load the credentials file for ansible server
          withCredentials([sshUserPrivateKey(credentialsId: 'ansible-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')]) {
            remote.user = user
            remote.identityFile = keyfile
            // initialize ansible server
            sshScript remote: remote, script: "prepare-ansible-server.sh"
            // execute playbook on ansible server
            sshCommand remote: remote, command: "ansible-playbook my-playbook.yaml"
          }
        }
      }
    }      
  }
} 
