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
    }
}
