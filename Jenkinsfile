//所有脚步命令都放在pipeline中
pipeline{
	//指定任务再哪个集群节点中执行
	agent any

	//声明全局变量,方便后面使用 
	environment {
        harborUser = 'gaoyanbing'
        harborPassword = 'Gyb-890702'
        harborAddress = 'docker.io'
        harborRepo = 'gaoyanbing'
	}

	stages {
		stage('拉取git仓库代码'){
			steps{
				checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'bb77c4d8-db23-4f5f-88e1-b0307a19ad08', url: 'https://gitee.com/gaoyanbing/mytest.git']]])
			}
		}
		stage('通过maven构建项目'){
			steps{
				sh '/var/jenkins_home/maven/bin/mvn clean package -DskipTests'
			}
		}
		stage('通过sonarqube对代码做质量检测'){
			steps{
				sh '/var/jenkins_home/sonar-scanner/bin/sonar-scanner  -Dsonar.sources=./ -Dsonar.projectname=${JOB_NAME} -Dsonar.projectKey=${JOB_NAME} -Dsonar.java.binaries=./target/ -Dsonar.login=124cbb8c84f62dd30601d8cdfe04aa9077249224'
			}
		}
		stage('通过docker制作自定义镜像'){
			steps{
				sh '''mv target/*.jar docker/
                                docker build -t ${JOB_NAME}:latest docker/'''
			}
		}
		stage('将自定义镜像推送到Harbor'){
			steps{
				sh '''docker login -u ${harborUser} -p ${harborPassword} ${harborAddress}
				docker tag ${JOB_NAME}:latest ${harborAddress}/${harborRepo}/${JOB_NAME}:latest
				docker push ${harborAddress}/${harborRepo}/${JOB_NAME}:latest'''
			}
		}
		stage('将yml文件传到k8s-master上'){
			steps{
                sshPublisher(publishers: [sshPublisherDesc(configName: 'k8s', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'pipeline.yml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
			}
		}
        stage('远程部署'){
			steps{
                sh 'ssh root@8.130.16.222  kubectl apply -f /usr/local/k8s/pipeline.yml '
			}
		}

	}

}
