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
                  sh "mvn sonar:sonar -Dsonar.host.url=http://192.168.88.130:9000 -Dsonar.login=65607ba9d0f54590cf55fe8e60134fb5e87c557d"
                }
            }
        }
        stage('Deploy') { 
            agent { node { label 'master' } }
            steps {
                dir(env.WORKSPACE){
                   sh 'sshpass -p 1qaz@WSX scp -o StrictHostKeychecking=no target/sample.jar root@192.168.88.130:/opt/ansible'
                   sh 'sshpass -p 1qaz@WSX scp -o StrictHostKeychecking=no deploy.sh root@192.168.88.130:/opt/ansible'
                   sh 'sshpass -p 1qaz@WSX ssh -o StrictHostKeychecking=no root@192.168.88.130 "bash /opt/ansible/deploy.sh"'
                   sh 'sleep 8s'
                }
            }
        }
        stage('Robot Framework') { 
            agent { node { label 'robot' } }
            steps {
               dir(env.WORKSPACE){
	                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'binbin', url: 'https://github.com/zbbkeepgoing/springboot-demo.git']]])
	                sh "pybot -d /opt/workspace/CI+Sonar+Sh+Robot+Jmeter robot/demo.robot"
	                step([$class: 'RobotPublisher',
	                  disableArchiveOutput: false,
	                  logFileName: 'log.html',
	                  otherFiles: '',
	                  outputFileName: 'output.xml',
	                  outputPath: '.',
	                  passThreshold: 40,
	                  reportFileName: 'report.html',
	                  unstableThreshold: 0]);
	            }
            }
        }
		stage('Jmeter') { 
            agent { node { label 'jmeter' } }
            steps {
                sh "rm -rf  /opt/workspace/CI+Sonar+Sh+Robot+Jmeter/jmeter/*"
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'binbin', url: 'https://github.com/zbbkeepgoing/springboot-demo.git']]])
                sh "mkdir -p /opt/workspace/CI+Sonar+Sh+Robot+Jmeter/jmeter/output"
                sh "jmeter.sh -n -t /opt/workspace/CI+Sonar+Sh+Robot+Jmeter/jmeter/demo.jmx  -l /opt/workspace/CI+Sonar+Sh+Robot+Jmeter/jmeter/demo.jtl -j /opt/workspace/CI+Sonar+Sh+Robot+Jmeter/jmeter/demo.log -e -o /opt/workspace/CI+Sonar+Sh+Robot+Jmeter/jmeter/output"
                step([$class: 'ArtifactArchiver', artifacts: 'jmeter/*,jmeter/output/*'])
                perfReport "jmeter/demo.jtl"
            }
        }
    }
}
