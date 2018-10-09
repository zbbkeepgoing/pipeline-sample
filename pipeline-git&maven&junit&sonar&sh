pipeline {
    agent none 
    stages {
       stage('Preparation') { 
            agent { node { label 'master' } }
            steps {
               checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'binbin', url: 'https://github.com/zbbkeepgoing/springboot-demo.git']]])
            }
        }
         
        stage('Build') { 
            agent { node { label 'master' } }
            steps {
                dir(env.WORKSPACE){
                  sh "mvn clean install"
                  junit allowEmptyResults: true, keepLongStdio: true, testResults: 'target/surefire-reports/*.xml'
                  sh "mv target/sample-0.0.1-SNAPSHOT.jar target/sample.jar"
                }
            }
        }
        stage('Sonarqube') { 
            agent { node { label 'master' } }
            steps {
                dir(env.WORKSPACE){
                  sh "mvn sonar:sonar -Dsonar.host.url=http://xxxx:9000 -Dsonar.login=sonarkey"
                }
            }
        }
        stage('Deploy') { 
            agent { node { label 'master' } }
            steps {
                dir(env.WORKSPACE){
                   sh 'sshpass -p 1qaz@WSX scp -o StrictHostKeychecking=no target/sample.jar root@xxxx:/opt/ansible'
                   sh 'sshpass -p 1qaz@WSX scp -o StrictHostKeychecking=no deploy.sh root@xxxx:/opt/ansible'
                   sh 'sshpass -p 1qaz@WSX ssh -o StrictHostKeychecking=no root@xxxx "bash /opt/ansible/deploy.sh"'
                   sh 'sleep 8s'
                }
            }
        }
    }
}
