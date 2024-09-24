------

# **Gitlab 和 Jenkins 的 CI CD 配置**



## 简介

CI CD 的概念：持续集成和持续交付

1. 自动化
2. 产物管理、备份
3. 提交代码紧接测试

https://docs.gitlab.cn/jh/ci/introduction/index.html

## JENKINS 与 GITLAB CICD的区别

|              | Jenkins                | Gitlab CICD          |
| ------------ | ---------------------- | -------------------- |
| 特点         | 中心化                 | 基于runner实现分布式 |
| 操作方式     | 用户图形界面、Pipeline | Pipeline             |
| 插件系统     | 丰富插件               | 几乎没有插件         |
| Pipeline语法 | groovy                 | yaml                 |
| Pipeline难度 | 简单                   | 简单                 |
| Docker支持   | 是                     | 是                   |
| 代码仓库集成 | 否                     | 是                   |
| 合并分支构建 | 否                     | 是                   |

## gitlab-runner 

https://docs.gitlab.cn/jh/ci/

http://10.5.0.71:8880/admin/runners

在极狐GitLab 中，runners 是运行 CI/CD job的代理。

### 安装

https://docs.gitlab.cn/runner/install/index.html

我是直接使用rpm包安装的。

###　注册 

http://10.5.0.71:8880/admin/runners

sudo gitlab-runner register --url http://10.5.0.71:8880/ --registration-token S4xsm3KMahvtEskAyhkc

![image-20240105183225196](./assets/image-20240105183225196.png)

**需要注意的是：一台主机可以注册多个runner，用于实现不同executor的组合**

### tags

标签用来匹配Pipeline中的job和runner。runner只会执行与自己标签匹配的job（也可以配置成执行无标签的job）。

```yaml
build:
  variables:
    GIT_STRATEGY: clone
  tags:
    - shell_solax04
  stage: build
  script:
    - echo "Hello, $GITLAB_USER_LOGIN!"
    #   MAVEN 打包命令 参数含义分别是:  跳过测试 \ 指定模块 user \ 编译指定模块的依赖 \ 输出完整DEBUG级别日志 -X
    - mvn clean package -DskipTest -pl user -am
    #    - mvn clean package -DskipTest
    - mv $TARGET_PATH/$TARGET_JAR ./
    #   修改目标JAR文件名: 项目名 + 模块名 + 分支 + Pipeline唯一ID (本次部署的唯一ID)
    - mv $TARGET_JAR ${PROJECT_NAME}-${TARGET_JAR:0:12}-$CI_COMMIT_BRANCH-$CI_PIPELINE_ID.jar
  artifacts:
    #   保留产物, 同一个PIPELINE的不同JOBS可以共享产物, 直接引用文件名即可
    paths:
      - ${PROJECT_NAME}-${TARGET_JAR:0:12}-$CI_COMMIT_BRANCH-$CI_PIPELINE_ID.jar
      - $TARGET_PATH/*
    expire_in: 1 month
```

### 配置

https://docs.gitlab.cn/runner/configuration/advanced-configuration.html

配置文件：/etc/gitlab-runner/config.toml

#### clone_url

https://docs.gitlab.cn/runner/configuration/advanced-configuration.html#clone_url-%E5%A6%82%E4%BD%95%E5%B7%A5%E4%BD%9C

**如果不配置clone_url，gitlab runner默认拉取代码会使用HTTPS加密端口，如果没有配置好证书，这会失败**

```toml
concurrent = 1
check_interval = 0
log_level = "debug"
shutdown_timeout = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "micro-cloud-package"
  url = "http://10.5.0.71:8880/"
  id = 4
  token = "NSQnngEnssrcU6Udysi4"
  token_obtained_at = 2023-12-20T11:44:52Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "shell"
  clone_url = "http://10.5.0.71:8880/"
  [runners.custom]
    run_exec = ""

[[runners]]
  name = "remote ssh"
  url = "http://10.5.0.71:8880"
  id = 5
  token = "oxiapBLrA39ToQnfSZRm"
  token_obtained_at = 2023-12-22T10:02:12Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "ssh"
  [runners.ssh]
    user = "root"
    host = "solax07"
    port = "22"
    identity_file = "/home/gitlab-runner/.ssh/id_rsa"

[[runners]]
  name = "runner in docker"
  url = "http://10.5.0.71:8880"
  id = 6
  token = "Dp1nYofa7pnYHju_sTpZ"
  token_obtained_at = 2023-12-25T05:26:49Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "docker"
  builds_dir = "/tmp/builds"
  clone_url = "http://10.5.0.71:8880/"
  [runners.cache]
    MaxUploadedArchiveSize = 0
  [runners.docker]
    tls_verify = false
    image = "docker:stable"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock", "/tmp/builds:/tmp/builds"]
    shm_size = 0
    network_mtu = 0
```

## gitlab-runner executor

https://docs.gitlab.cn/runner/executors/

注册一个runner，需要选择一种executor。常用的executor是shell docker ssh。也可以自定义。比起自定义，**更好用的做法是在同一个Pipeline中，组合不同的runner，完成不同的job**

1. shell：执行linux本机的命令（使用本机的环境）
2. ssh：执行linux远程主机的命令（使用远程主机的环境）
3. docker：拉取镜像，配置环境，执行命令，无需本地配置环境，同时gitlab有集中的缓存机制，可以保存使用过的镜像。

## gitlab ci cd Pipeline

### variables

https://docs.gitlab.cn/jh/ci/variables/  有三个作用

1. 定义变量
2. 使用预定义变量
3. 修改配置

另外，变量分全局变量和job变量。job优先使用job变量。



```yaml
variables:
  PROJECT_NAME: micro-cloud
    #  目标JAR文件名
  TARGET_JAR: user-execute.jar
```



```shell
#   修改目标JAR文件名: 项目名 + 模块名 + 分支 + Pipeline唯一ID (本次部署的唯一ID)
mv $TARGET_JAR ${PROJECT_NAME}-${TARGET_JAR:0:12}-$CI_COMMIT_BRANCH-$CI_PIPELINE_ID.jar

-rw-r----- 1 root root 57209307 Dec 25 11:38 micro-cloud-user-execute-v5.1-92.jar
-rw-r----- 1 root root 57209302 Dec 25 11:46 micro-cloud-user-execute-v5.1-93.jar
-rw-r----- 1 root root 57209303 Dec 25 11:57 micro-cloud-user-execute-v5.1-94.jar
-rw-r----- 1 root root 57209544 Dec 25 12:13 micro-cloud-user-execute-v5.1-95.jar
-rw-r----- 1 root root 57211039 Dec 26 09:35 micro-cloud-user-execute-v5.1-96.jar
-rw-r----- 1 root root 57211038 Dec 26 10:45 micro-cloud-user-execute-v5.1-97.jar
-rw-r----- 1 root root 57211037 Dec 26 11:00 micro-cloud-user-execute-v5.1-98.jar
-rw-r----- 1 root root 57211037 Dec 26 11:07 micro-cloud-user-execute-v5.1-99.jar
-rw------- 1 root root   109167 Dec 26 13:59 nohup.out
-rwxr-xr-x 1 root root      342 Dec 26 13:30 start.sh
```

#### 修改git策略可以使用变量 GIT_STRATEGY 

https://docs.gitlab.com/ee/ci/runners/configure_runners.html#git-strategy

There are three possible values: `clone`, `fetch`, and `none`. If left unspecified, jobs use the [project’s pipeline setting](https://docs.gitlab.com/ee/ci/pipelines/settings.html#choose-the-default-git-strategy).

`clone` is the slowest option. It clones the repository from scratch for every job, ensuring that the local working copy is always pristine. If an existing worktree is found, it is removed before cloning.

`fetch` is faster as it re-uses the local working copy (falling back to `clone` if it does not exist). `git clean` is used to undo any changes made by the last job, and `git fetch` is used to retrieve commits made after the last job ran.

However, `fetch` does require access to the previous worktree. This works well when using the `shell` or `docker` executor because these try to preserve worktrees and try to re-use them by default.

```yaml
variables:
    GIT_STRATEGY: clone
# GIT_STRATEGY 可选参数 clone fetch none，
```

**CENTOS7系统自带的git版本太旧，不能使用fetch**

#### 开启DEBUG模式 CI_DEBUG_TRACE

https://docs.gitlab.cn/jh/ci/variables/#%E5%90%AF%E7%94%A8-debug-%E6%97%A5%E5%BF%97

#### 变量共享

http://10.5.0.71:8880/groups/solax-cloud/-/settings/ci_cd

### 产物

https://docs.gitlab.cn/jh/ci/caching/#%E4%BA%A7%E7%89%A9

http://10.5.0.71:8880/solax-cloud/micro-cloud/-/artifacts

产物包括job过程中一切的可以保留的文件。

1. 产物可以设置过期时间，并且由gitlab统一管理，有效管理备份jar，还可以在gitlab页面管理手动管理产物；
2. 同意Pipeline中的job共享产物，共享产物的方法很简单，直接使用路径，搭配通配符可以保存文件夹内的所有文件。

### 缓存

https://docs.gitlab.cn/jh/ci/caching/#%E7%BC%93%E5%AD%98

使用 `cache` 关键字定义每个job的缓存。否则它被禁用。

后续Pipeline可以使用缓存。

如果依赖项相同，同一Pipeline中的后续job可以使用缓存。

不同的项目不能共享缓存

**缓存最大的用处是缓存依赖项，例如 maven,  npm, docker image**

## Pipeline

1. 页面编辑：http://10.5.0.71:8880/solax-cloud/micro-cloud/-/ci/editor?branch_name=v5.1
2. IDE本地编辑：项目根目录创建文件：.gitlab-ci.yml

### 结构

一个Pipeline由多个job组成，每个job包括要执行的命令。job通过tags与runner匹配。

### 过程

job是商品，runner是消费者。

一次代码提交根据Pipeline定义产生job队列，由消费者消费。

------

# **SONARQUBE集成**

静态代码检查工具，支持多重语言  http://10.5.0.71:9000/projects

sonarqube可以直接跟gitlab集成，https://docs.sonarsource.com/sonarqube/latest/devops-platform-integration/gitlab-integration/

**配置**：项目根目录添加 sonar-project.properties 文件（也可以作为 sonar-scanner 命令后的参数）

```properties
sonar.projectKey=solax-cloud_micro-cloud_AYy1Ae_QcAfxuEdxdOX2
sonar.qualitygate.wait=true
sonar.java.binaries=./user/target
```

---------

# 示例

## 后端

```yaml
# 全局变量
variables:
  PROJECT_NAME: micro-cloud
  #  目标路径
  TARGET_PATH: ./user/target
  #  目标JAR文件名
  TARGET_JAR: user-execute.jar
  #  部署主机
  DEPLOY_HOST: root@solax07
  #  部署路径
  DEPLOY_PATH: /solax/application/jar/micro-cloud
  #  GIT 策略，拉取代码用fetch还是clone，默认是fetch。
  #  centos 7自带的git版本太旧，用fetch会报错。
  #  全局默认不用拉取代码
  GIT_STRATEGY: none
  #  DEBUG
  CI_DEBUG_TRACE: "true"
  #  DEBUG
  CI_DEBUG_SERVICES: "false"
  SONARQUBE_SCAN_DIR: user

stages:
  - build
  - sonarqube-check
  - upload
  - deploy


# MAVEN PACKAGE
build:
  variables:
    GIT_STRATEGY: clone
  tags:
    - shell_solax04
  stage: build
  script:
    - echo "Hello, $GITLAB_USER_LOGIN!"
    #   MAVEN 打包命令 参数含义分别是:  跳过测试 \ 指定模块 user \ 编译指定模块的依赖 \ 输出完整DEBUG级别日志 -X
    - mvn clean package -DskipTest -pl user -am
    #    - mvn clean package -DskipTest
    - mv $TARGET_PATH/$TARGET_JAR ./
    #   修改目标JAR文件名: 项目名 + 模块名 + 分支 + 流水线唯一ID (本次部署的唯一ID)
    - mv $TARGET_JAR ${PROJECT_NAME}-${TARGET_JAR:0:12}-$CI_COMMIT_BRANCH-$CI_PIPELINE_ID.jar
  artifacts:
    #   保留产物, 同一个PIPELINE的不同JOBS可以共享产物, 直接引用文件名即可
    paths:
      - ${PROJECT_NAME}-${TARGET_JAR:0:12}-$CI_COMMIT_BRANCH-$CI_PIPELINE_ID.jar
      - $TARGET_PATH/*
    expire_in: 1 month

sonarqube-check:
  tags:
    - docker_solax04
  stage: sonarqube-check
  image:
    name: sonarsource/sonar-scanner-cli:5.0
    entrypoint: [""]
  variables:
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  # Defines the location of the analysis task cache
    GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script:
    - pwd
    - ls -la
    - sonar-scanner -X
  allow_failure: true
#  only:
#    - merge_requests
#    - master
#    - main
#    - develop


#  上传JAR包
upload:
  stage: upload
  tags:
    - shell_solax04
  script:
    - scp ${PROJECT_NAME}-${TARGET_JAR:0:12}-$CI_COMMIT_BRANCH-$CI_PIPELINE_ID.jar $DEPLOY_HOST:$DEPLOY_PATH

#  杀死上一个进程，并执行新的JAR包
deploy:
  tags:
    - ssh_solax07 
  stage: deploy
  script:
#    找到之前的JAR包进程并杀死（可能会报错）
    - ps -ef | grep ${PROJECT_NAME}-${TARGET_JAR:0:12} | grep -v grep | awk '{print $2}' | xargs kill -9
#    after_script 相当于 try catch 中的 finally
  after_script:
    - cd $DEPLOY_PATH
    - "nohup java -Xms1024m -Xmx4096m -jar ${PROJECT_NAME}-${TARGET_JAR:0:12}-$CI_COMMIT_BRANCH-$CI_PIPELINE_ID.jar  -Dspring.cloud.bootstrap.location=bootstrap.yml >> log 2>&1 &"
```

## 前端

```yaml
stages:          # List of stages for jobs, and their order of execution
  - sonarqube-check


sonarqube-check:
  tags:
    - docker_solax04
  stage: sonarqube-check
  image: 
    name: sonarsource/sonar-scanner-cli:5.0
    entrypoint: [""]
  variables:
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"  # Defines the location of the analysis task cache
    GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
    GIT_STRATEGY: clone
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script: 
    - sonar-scanner -X
  allow_failure: true
```

## JENKINS

```groovy
pipeline {
    agent any

    environment {
        project_name = "solax-cloud-installerapp"
        project_path = "/solax/application/jar/installerApp"
        project_scm_url = "https://10.5.0.68/svn/solax/NEW_PROJECT/户用型项目/srcs/trunk/solax-cloud"
        scm_credential = "57853bfb-c533-47c9-b1fa-f03716a2c769"
        publish_host = "10.5.0.45"
        publish_host_name = "solax05"
        publish_host_ssh_username = "root"
        publish_host_ssh_password = "al20160826@solax"
    }


    stages {
        stage("Checkout"){
            steps {
                checkout([$class: 'SubversionSCM', additionalCredentials: [], excludedCommitMessages: '', excludedRegions: '', excludedRevprop: '', excludedUsers: '', filterChangelog: false, ignoreDirPropChanges: false, includedRegions: '', locations: [[cancelProcessOnExternalsFail: true, credentialsId: '57853bfb-c533-47c9-b1fa-f03716a2c769', depthOption: 'infinity', ignoreExternalsOption: true, local: '.', remote: 'https://10.5.0.68/svn/solax/NEW_PROJECT/户用型项目/srcs/trunk/solax-cloud']], quietOperation: true, workspaceUpdater: [$class: 'UpdateUpdater']])
            }
        }

        stage("Build"){
            steps {
                withMaven(globalMavenSettingsConfig: 'e05f7d84-ebf9-40cb-b449-8c3d377a770b', jdk: 'java8', maven: 'maven39', traceability: true) {
                    sh "mvn package -DskipTests -pl ${project_name} -am"
                    sh "ls"
                    sh "mv ${project_name}/target/${project_name}-exec.jar ${project_name}/target/${project_name}.jar"
                    sh "mv ${project_name}/target/${project_name}.jar ./"
                }
            }
        }

        stage("Archive"){
            steps {
                script {
                    def remote = [:]
                    remote.name = "${publish_host_name}"
                    remote.host = "${publish_host}"
                    remote.user = "${publish_host_ssh_username}"
                    remote.password = "${publish_host_ssh_password}"
                    remote.allowAnyHosts = true
                    sshCommand remote: remote, command: "if [ -f \"${project_path}/${project_name}.jar\" ]; then \n mv ${project_path}/${project_name}.jar ${project_path}/${project_name}.jar_\$(date \"+%Y%m%d%H%M\"); \n fi"
                    sshPut remote: remote, from: "${project_name}.jar", into: "${project_path}"
                }
            }
        }

        stage("Deliver"){
            steps {
                script {
                    def remote = [:]
                    remote.name = "${publish_host_name}"
                    remote.host = "${publish_host}"
                    remote.user = "${publish_host_ssh_username}"
                    remote.password = "${publish_host_ssh_password}"
                    remote.allowAnyHosts = true
                    writeFile file: "installer_app_start.sh", text: """
pid=\$(ps -ef | grep solax-cloud-installerapp.jar | grep -v grep | awk \'{print \$2}\')
if [ ! -n "\$pid" ]; then
    echo "no such process";
else
    echo "killing process";
    kill -9 \$pid;
fi
cd ${project_path};
source /etc/profile;
nohup java -jar -Dspring.profiles.active=test solax-cloud-installerapp.jar > \$(date \"+%Y%m%d%H%M\").log 2>&1 &
"""
                    sh "cat installer_app_start.sh"
                    sshScript remote: remote, script: "installer_app_start.sh"
                }
            }
        }
    }

    post {
        always { 
            archiveArtifacts artifacts: 'solax-cloud-installerapp.jar', followSymlinks: false
            cleanWs()
        }
    }
}
```

# 其他

由于现在的gitlab实例用了docker转发8880端口到80端口，4443端口转发到443端口，而gitlab页面有些点击项是不带端口的。直接点击会跳转失败。
