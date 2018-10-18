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
                  sh "mvn sonar:sonar -Dsonar.host.url=http://192.168.88.130:9000 -Dsonar.login=2382ac098363521b98731e286e52e1ad22adef2b"    //指定sonar的ip和token
                }
            }
        }
        stage('Deploy') { 
            agent { node { label 'master' } }
            steps {
                dir(env.WORKSPACE){
                   sh 'sshpass -p 1qaz@WSX scp -o StrictHostKeychecking=no target/sample.jar root@192.168.88.130:/opt/ansible'   //把部署的jar传到目标机器上
                   sh 'sshpass -p 1qaz@WSX scp -o StrictHostKeychecking=no deploy.sh root@192.168.88.130:/opt/ansible'   //把脚本传到目标机器上
                   sh 'sshpass -p 1qaz@WSX ssh -o StrictHostKeychecking=no root@192.168.88.130 "bash /opt/ansible/deploy.sh"'      //在目标机器上执行对应的脚本
                   sh 'sleep 8s'     //睡眠8s
                }
            }
        }
        stage('Robot Framework') { 
            agent { node { label 'robot' } }   //指定label为robot节点，即自动化测试节点
            steps {
               dir(env.WORKSPACE){
	                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'binbin', url: 'https://github.com/zbbkeepgoing/springboot-demo.git']]])    //拉去最新工程，因为自动化测试脚本在工程中
	                sh "pybot -d /opt/workspace/CI+Sonar+Sh+Robot robot/demo.robot"  //生成报告路径需要注意
	                step([$class: 'RobotPublisher',     //将自动化测试的报告推送给jenkins服务
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
    }
}
