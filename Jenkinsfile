pipeline{
      // 定义groovy脚本中使用的环境变量
      environment{
        // 本示例中使用DEPLOY_TO_K8S变量来决定把应用部署到哪套容器集群环境中，如“Production Environment”， “Staging001 Environment”等
        IMAGE_TAG =  sh(returnStdout: true,script: 'echo $image_tag').trim()
        // 镜像仓库地址
        ORIGIN_REPO =  sh(returnStdout: true,script: 'echo $origin_repo').trim()
        // 镜像仓库名称
        REPO =  sh(returnStdout: true,script: 'echo $repo').trim()
        // gitlab revision用于滚动更新镜像
        REVISION =  sh(returnStdout: true,script: 'echo $revision').trim()
        // 项目名称
        PROJECT_NAME = sh(returnStdout: true,script: 'echo $project_name').trim()
      }

      // 定义本次构建使用哪个标签的构建环境，本示例中为 “slave-pipeline”
      agent{
        node{
          label 'slave-pipeline'
        }
      }

      // "stages"定义项目构建的多个模块，可以添加多个 “stage”， 可以多个 “stage” 串行或者并行执行
      stages{
        // 定义第一个stage， 完成克隆源码的任务
        // stage('Git'){
        //   steps{
        //     sh "git config --global http.sslVerify false"
        //     sh "git version"
        //     git branch: '${BRANCH}', credentialsId: 'wangdi-gitlab-password', url: 'https://39.108.48.66:40081/wangdi/nfplus-checher.git'
        //   }
        // }

    
        // 添加第四个stage, 运行容器镜像构建和推送命令， 用到了environment中定义的groovy环境变量
        stage('Image Build And Publish'){
          steps{
              container("kaniko") {
                  sh "kaniko -f `pwd`/Dockerfile -c `pwd` --destination=${ORIGIN_REPO}/${REPO}:${IMAGE_TAG}"
              }
          }
        }


        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    step([$class: 'KubernetesDeploy', authMethod: 'certs', apiServerUrl: 'https://kubernetes.default.svc.cluster.local:443', credentialsId:'k8sCertAuth', config: 'deployment.yaml',variableState: 'ORIGIN_REPO,REPO,IMAGE_TAG,REVISION,PROJECT_NAME'])
                }
            }
        }
      }
    }
