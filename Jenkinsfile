pipeline {
  agent any
  stages {
    stage('检出') {
      steps {
        checkout(scm: [
          $class: 'GitSCM', branches: [[name: env.GIT_BUILD_REF]],
          userRemoteConfigs: [[
            url: env.GIT_REPO_URL,
            credentialsId: env.CREDENTIALS_ID
          ]]
        ], changelog: false)
      }
    }
    stage('install') {
      steps {
        sh 'npm install'
      }
    }
    stage('构建') {
      steps {
        sh 'npm run build'
        sh 'npm install -g hexo-cli'
        sh 'hexo clean'
        sh 'hexo g'
      }
    }
    stage('测试') {
      steps {
        sh 'ls -l'
      }
    }
    stage('部署') {
      steps {
        script {
          def remoteConfig = [:]
          remoteConfig.name = "my-remote-server"
          remoteConfig.host = "${env.REMOTE_HOST}"
          remoteConfig.allowAnyHosts = true

          // 使用当前项目下的凭据管理中的 用户名 + 密码 凭据
          withCredentials([usernamePassword(
            credentialsId: "${env.REMOTE_CRED}",
            passwordVariable: 'password',
            usernameVariable: 'userName'
          )]) {

            // SSH 登陆用户名
            remoteConfig.user = userName
            // SSH 登陆密码
            remoteConfig.password = password

            // 本地创建一个 test.sh 脚本，用来发送到远端执行
            sh 'ls -l'

            sshRemove (remote: remoteConfig, path: '/home/nginx/www/public/')
            sshPut (remote: remoteConfig, from: './public/', into: '/home/nginx/www/')
          }
        }

      }
    }
  }
}