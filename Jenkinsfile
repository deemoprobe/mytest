pipeline {
    agent any
    environment{
        harborHost = 'harbor.deemoprobe.com'
        harborRepo = 'devops'
        harborUser = 'admin'
        harborPasswd = 'Harbor12345'
        projectName = 'mytest'
    }

    // 存放所有任务的合集
    stages {

        stage('拉取Git代码') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "*/main"]], 
                extensions: [], 
                submoduleCfg: [], 
                userRemoteConfigs: [[url: "http://192.168.43.110:8929/root/mytest.git"]]])
            }
        }

        stage('构建代码') {
            steps {
                sh '/var/jenkins_home/maven/bin/mvn clean package -DskipTests'
            }
        }

        stage('制作自定义镜像并发布Harbor') {
            steps {
                sh '''cp ./target/*.jar ./docker/
                cd ./docker
                docker build -t ${projectName}:latest ./'''

                sh '''docker login -u ${harborUser} -p ${harborPasswd} ${harborHost}
                docker tag ${projectName}:latest ${harborHost}/${harborRepo}/${projectName}:latest
                docker push ${harborHost}/${harborRepo}/${projectName}:latest'''
            }
        }

        stage('在Kubernetes中部署项目') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'k8s', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'kubectl apply -f /devops/pipeline.yaml && kubectl rollout restart deployment -n web myapp', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'pipeline.yaml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
    }
    post {
        success {
            dingtalk (
                robot: 'JenkinsRobot',
                type:'MARKDOWN',
                title: "success: ${projectName}",
                text: ["- 构建成功:${projectName}项目!\n- 版本:latest\n- 持续时间:${currentBuild.durationString}\n- 任务:#${projectName}"]
            )
        }
        failure {
            dingtalk (
                robot: 'JenkinsRobot',
                type:'MARKDOWN',
                title: "fail: ${projectName}",
                text: ["- 构建失败:${projectName}项目!\n- 版本:latest\n- 持续时间:${currentBuild.durationString}\n- 任务:#${projectName}"]
            )
        }
    }
}
