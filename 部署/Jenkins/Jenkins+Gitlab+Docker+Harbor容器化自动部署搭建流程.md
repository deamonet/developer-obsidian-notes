# Jenkins+Gitlab+Docker+Harbor容器化自动部署搭建流程

本文参考：https://www.cnblogs.com/incognitor/p/16603106.html

## 前期准备

关闭防火墙：systemctl stop firewalld  systemctl disable firewalld

环境：Linux版本：CentOs7

- 更新yum源

yum update

- 安装docker

yum install docker -y

- 启动docker

systemctl start docker

- 开启docker远程访问

*vim* /lib/systemd/system/docker.service

\#添加  -H tcp://0.0.0.0:2375

![img](media/img-2.png)

## 安装gitlab

1.docker拉取镜像（社区版）

docker pull gitlab/gitlab-ce

------

2.创建本地文件夹

mkdir -p /home/local/gitlab_docker/gitlab

mkdir -p /home/local/gitlab_docker/logs

mkdir -p /home/local/gitlab_docker/data

------

3.运行gitlab镜像

docker run -d -p 8443:443 -p 8090:80 -p 8022:22 --name gitlab --restart always -v /home/local/gitlab_docker/gitlab:/etc/gitlab -v /home/local/gitlab_docker/logs:/var/log/gitlab -v /home/local/gitlab_docker/data:/var/opt/gitlab gitlab/gitlab-ce

参数说明：

-d：后台运行

-p：端口映射，宿主机端口：容器端口

--name： 给将要运行的容器命名

--restart always：docker启动的时候，也自行启动

-v：挂载目录，宿主机目录：容器目录

gitlab/gitlab-ce：要运行的镜像

------

4.修改gitlab.rb配置文件

vim /home/local/gitlab_docker/gitlab/gitlab.rb

内容如下：
\##改成本机ip
external_url 'http://192.168.42.227'
gitlab_rails['gitlab_ssh_host'] = '192.168.42.227'

\##上面映射的端口
gitlab_rails['gitlab_shell_ssh_port'] = 8022

------

5.进入容器重启配置

\##进去gitlab容器的命令
docker exec -it gitlab bash

\##重置gitlab客户端的命令
gitlab-ctl reconfigure

------

6.修改http的clone地址加上端口

\##进入容器内部
docker exec -it gitlab /bin/bash

\##修改文件
vi /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml

内容如下：

\##改成本机ip
host: 192.168.42.227
\##clone的端口，上面映射的端口
port: 8090
https: false

------

7.重启gitlab,在容器内执行

gitlab-ctl restart

------

8.初始账号密码

账号：root

密码：进入到容器执行 cat /etc/gitlab/initial_root_password

注解：如果使用docker restart gitlab，会自动执行gitlab-ctl reconfigure，配置会被还原，我们自己刚刚改的配置会丢失，慎用docker restart gitlab

## 安装Harbor

1.安装docker-compose

下载地址：https://github.com/docker/compose/releases ，选中docker-compose-linux-x86_64下载

------

2.下载完成重命名为：docker-compose，并放到 /usr/local/bin/下

------

3.赋予可执行权限

sudo chmod +x /usr/local/bin/docker-compose

------

4.创建软连接

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

------

5.检查安装结果

docker-compose --version

------

6.下载harbor

https://github.com/goharbor/harbor/releases/download/v2.4.1/harbor-offline-installer-v2.4.1.tgz

并放到目录下： /home/local/

------

7.解压文件

tar xzvf harbor-offline-installer-v2.4.1.tgz

------

8.拷贝配置文件

cp /home/local/harbor/harbor.yml.tmpl /home/local/harbor/harbor.yml

------

9.修改http访问地址,禁用https

vi /home/local/harbor/harbor.yml

内容如下：

\#本机ip
hostname: 192.168.42.227
http:
	port: 8080

\#https 注解https下面的所属配置

\# https related config

\#https:

  \# https port for harbor, default is 443

 \# port: 443

  \# The path of cert and key files for nginx

  \#certificate: /your/certificate/path

  \#private_key: /your/private/key/path



------

10.执行安装

sudo /home/local/harbor/install.sh

------

11.启动、关闭：在harbor目录下执行命令

启动：

docker-compose up -d

关闭：

docker-compose down

------

12.访问web

（ip:8080）默认账号密码：admin     Harbor12345

------

如果关闭了防火墙之后，需要重启服务器，harbor才能启动。

------

## Harbor报错

### Error response from daemon: OCI runtime create failed: runc create failed: container with given ID already exists: unknown

![img](media/img.png)

解决：

cd /run/docker/runtime-runc/moby

ls

![img](media/img-1.png)

删除即可。

### Error response from daemon: failed to initialize logging driver: dial tcp [::1]:1514: connect: connection refused

解决：

- 停止harbor服务  docker-compose stop
- vim /etc/rsyslog.conf

取消注释并修改

$ModLoad imtcp

$InputTCPServerRun 1514

![img](media/img-7.png)

- 重启rsyslog服务  systemctl restart rsyslog.service
- 再次启动harbor  docker-compose up -d

如果还是报一样的错，可以尝试重启服务器，然后再次启动或者直接docker-compose restart。

------

## 为docker设置harbor私服

1.将harbor私服的http地址配置到docker的不安全的register中，修改配置文件

vim /etc/docker/daemon.json，

追加一行（和上一行用逗号隔开并回车）："insecure-registries": ["192.168.42.227:8080"]

------

2.重启docker

systemctl daemon-reload && systemctl restart docker

------

如果重启docker报错

![img](media/img-6.png)

关闭防火墙即可：systemctl stop firewalld  systemctl disable firewalld

3.docker登录（会显示登录成功）

docker login -u admin -p Harbor12345 http://192.168.42.227:8080

------

4.上传镜像到私服

在harbor私服新建一个项目名A，

按照命名规则创建一个镜像：

docker tag 本机镜像:tag harbor私服地址:端口/项目名A/文件夹:tag

例如：把本机java镜像上传到私服 test_public 目录下

docker tag java:8 192.168.42.227:8080/test_public/image_test:v1

------

5.上传到私服：docker push 私服地址/项目名/文件夹:Tags

登录：

docker login -u admin -p Harbor12345 http://192.168.42.227:8080

上传：

docker push 192.168.42.227:8080/test_public/image_test:v1

------

6.拉取私服镜像：docker pull 私服地址/项目名/文件夹:Tags

docker pull 192.168.42.227:8080/test_public/image_test:v1

## 安装Jenkins

（建议不要用docker安装，因为用docker安装Jenkins写pipeline脚本时，就用不了宿主机的插件及命令）

1.依次执行以下命令

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
 sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key

或者

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo   

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key


 sudo yum upgrade

\#JDK尽量不要使用yum安装，使用本地安装。
 sudo yum install java-11-openjdk
 sudo yum install jenkins

------

如果上面的yum中没有Jenkins安装

https://blog.csdn.net/qiuyeyijian/article/details/104570642

添加本地JDK路径，该路径和Jenkins web页面配置的一样就行

- vim /etc/init.d/jenkins

![img](media/img-3.png)

------

2.修改jenkins用户

vi /etc/sysconfig/jenkins

JENKINS_USER="jenkins" 改成 JENKINS_USER="root"

------

3.配置

重新加载配置：
sudo systemctl daemon-reload

设置开机启动、关闭（先直接跳过该操作）

启动：

sudo systemctl enable jenkins

关闭：

sudo systemctl disable jenkins

------

4.启动

systemctl start jenkins

------

5.如果在上一步启动成功了，可忽略此步

如果配置无误，启动还是报错，可以换个启动方法（本人就是这样启动成功的）

先关闭Jenkins，即使启动失败，也不代表是关闭的：

systemctl stop jenkins

注：如关闭不成功就重启Liunx，确认已经关闭了jenkins

进入目录：

cd /etc/init.d/

启动：

./jenkins start

关闭命令是：./jenkins stop 

如果启动失败，可以查看日志：/var/log/jenkins/jenkins.log

如果是端口被占用：可以使用ps -ef | grep jenkins查看，然后kill -9

------

如果遇见报错

![img](media/img-1.png)

yum -y install fontconfig





6.web访问，根据界面提示获取密码

cat /var/lib/jenkins/secrets/initialAdminPassword

------

7.安装社区推荐的插件

------

8.等待安装完成，进入Jenkins主页

------

## 配置Jenkins

1.下载maven：https://maven.apache.org/download.cgi  下载 apache-maven-3.8.6-bin.tar.gz

------

2.放到 /home/local 目录下，并进行解压

tar xzvf apache-maven-3.8.6-bin.tar.gz

------

3.修改配置源：

<mirror>
<id>nexus-aliyun</id>
<mirrorOf>central</mirrorOf>
<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
</mirror>

------

4.修改环境变量（追加以下内容）：vim /etc/profile

export PATH=${PATH}:/home/local/apache-maven-3.8.6/bin

------

5.重载环境变量

source /etc/profile

------

6.Dashboard -> Manage Jenkins -> Global Tool Configuration

Maven 配置:将centos中maven目录拷贝进去

注意：在构建调用mvn报权限，执行命令：chmod a+x /目录/apache-maven-3.2.2/bin/mvn

![img](media/img-4.png)

- JDK配置：将centos的JDK目录拷贝进去，可以通过命令：cat /etc/profile   查看java的目录

![img](media/img-8.png)

- Git配置

先在centos安装git：

yum install git -y

![img](media/img-6.png)

- Maven配置

![img](media/img-3.png)

- 保存，其他插件配置忽略

## 安装插件：Git Parameter

Dashboard -> Manage Jenkins -> 插件管理    处输入Git Parameter进行安装，安装完成重启Jenkins

该插件用于在Jenkins拉取gitlab代码时，可以选择拉取不同分支的代码

## 安装插件：Publish Over SSH

\#配置

Dashboard -> Manage Jenkins -> Configure System -> 滑到最下面

![img](media/img-2.png)

![img](media/img-7.png)

------

## 配置一个自动部署的项目

- 新建一个item，输入项目名称，选择中pipeline（流水线，如果没有该选项，就去安装该插件，并重启），点击确定保存

![img](media/img-5.png)

- 配置《丢弃旧构建》策略

![img](media/img-2.png)

- 配置项目参数：新增 String Parameter，用于在构建项目时，可输入一个字符参数Tag，在写pipeline script时，可以通过${Tag}来引用

![img](media/img-6.png)

- 再新添一个项目参数：Git Parameter（刚刚安装Git Parameter插件才有该选项），用于在构建时可选择不同的分支拉取代码进行构建，在写pipeline脚本时通过 ${branch} 引用该值

注意：分支过滤：.*  是默认值，故${branch}=origin/main ，我改成了：origin.*/(.*)   ${branch}=main，在pipeline脚本里取值 ${branch}拉取代码就不会报错了。其他配置默认值

![img](media/img-1.png)

- 编写pipeline脚本

![img](media/img-5.png)

- 拉取gitlab代码的脚本，可以通过点击《流水线语法》进行自动生产

![img](media/img-7.png)

- 无Credentials，点击添加：输入gitlab用户名和密码，点击保存即可

![img](media/img-3.png)

- 然后选中该凭证，点击生成流水线脚本，把脚本拷贝到pipeline脚本里

![img](media/img-6.png)

- 例如

```plain
node {
　　　　stage('Preparation') {
　　　　　　checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: 'ecf3bed1-f362-4884-b285-fb42ab2d9816', url: 'http://192.168.42.227:8090/root/springboot.git']]])
　　　　　　echo '拉取代码成功'
　　　　}
　　}
```

- 然后，将springboot项目用maven进行打包成jar（跳过了测试），脚本如下

```plain
node {
　　　　stage('Preparation') {
　　　　　　checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: 'ecf3bed1-f362-4884-b285-fb42ab2d9816', url: 'http://192.168.42.227:8090/root/springboot.git']]])
　　　　　　echo '拉取代码成功'
　　　　}
　　　　stage ('maven build'){
　　　　　　sh ''' /home/local/apache-maven-3.8.6/bin/mvn clean package -DskipTests '''
　　　　　　echo '构建成功'
　　　　}
　　}
```

- 最后将jar打包成镜像，并上传到harbor私服的脚本：点击流水线语法，自动生成harbor凭证脚本

![img](media/img-7.png)

- 输入如下图，点击《添加》

![img](media/img-4.png)

- 输入账号密码，点击添加

![img](media/img-5.png)

- 点击生成流水线脚本，拷贝脚本

![img](media/img-7.png)

- 然后在 //some block 写自己的脚本，例如

```plain
node {
 
　　　　　stage('Preparation') {
　　　　　　　checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: 'ecf3bed1-f362-4884-b285-fb42ab2d9816', url: 'http://192.168.42.227:8090/root/springboot.git']]])
　　　　　　　echo '拉取代码成功'
　　　　　}
　　　　　stage ('maven build'){
　　　　　　　sh ''' /home/local/apache-maven-3.8.6/bin/mvn clean package -DskipTests '''
　　　　　　　echo '构建成功'
　　　　　}
 
　　　　　stage ('docker build'){
　　　　　　　withCredentials([usernamePassword(credentialsId: '1a5a43b8-4c27-47f6-95ef-af578568aa3e', passwordVariable: 'password', usernameVariable: 'username')]) {
　　　　　　　　　sh '''
　　　　　　　　　REPOSITORY=192.168.42.227:8080/test_public/image_test:${Tag}
　　　　　　　　　docker build -f Dockerfile -t $REPOSITORY .
　　　　　　　　　docker login -u ${username} -p ${password} http://192.168.42.227:8080
　　　　　　　　　docker push $REPOSITORY
　　　　　　　　　docker images | grep 'image_test'| awk '{print $3}'|xargs docker rmi
　　　　　　　　　'''
　　　　　　　}
　　　　　　　echo '镜像上传成功'
　　　　　}
　　　　}
```

- 说明

批量删除REPOSITORY包含”image_test“的镜像docker images | grep 'image_test'| awk '{print $3}'|xargs docker rmi<br><em id="__mceDel">

## 部署项目

- 首次构建的时候，可能不会显示分支branch，是空的，首次构建拉取代码成功了，就可以显示branch了，前提要保证：branch即使是空的，默认值是存在的分支，在git的流水线自动生成pipeline脚本时，配置好默认的分支

![img](media/img-7.png)

------

## 如果服务器异常关机导致jenkins起不来

进入到/etc/init.d目录

先执行：./jenkins stop

再执行：./jenkins start

## 使用jenkinsfile的方式读取文件

![img](media/img-5.png)

## 注意

- 项目中需要加上插件

```plain
<plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>1.4.13</version>
                <configuration>
                    <repository>${project.artifactId}</repository>
                    <buildArgs>
                        <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                    </buildArgs>
                </configuration>
            </plugin>
```

- 重启docker的时候先将harbor停止

在harbor目录执行 docker-compose stop

然后再重启docker

systemctl daemon-reload 

systemctl restart docker

## Dockerfile文件

- 在项目根目录新建Dockerfile文件

```plain
# 拉取jdk8作为基础镜像
FROM openjdk:8-jdk-alpine
# 输入参数，改参数为打包后的jar包，例如 api_gateway/target/api_gateway-1.0-SNAPSHOT.jar
ARG JAR_FILE
# 添加jar到镜像并命名为app.jar, copy命令是把 api_gateway-1.0-SNAPSHOT.jar直接添加到镜像里面不解压，ADD是添加入镜像并且解压
COPY ${JAR_FILE} chitang-mail.jar
# 镜像启动后暴露的端口
EXPOSE 5429
# 防止字体缺失
# RUN apk add --update ttf-dejavu fontconfig
#RUN echo -e 'https://mirrors.aliyun.com/alpine/v3.6/main/\nhttps://mirrors.aliyun.com/alpine/v3.6/community/' > /etc/apk/repositories \
 #&& apk update \
#&& apk upgrade \
 #&& apk --no-cache add ttf-dejavu fontconfig
# jar运行命令，参数使用逗号隔开
ENTRYPOINT ["java","-Duser.timezone=GMT+8","-jar","/chitang-mail.jar"]
#ENTRYPOINT ["java","-Duser.timezone=GMT+8","-jar","/chitang-mail.jar","--logging.config=classpath:logback-spring.xml"]
```

## 报错：

![img](media/img-6.png)

把这里改成服务器的密码！！

![img](media/img-2.png)

## 脚本

```plain
def harbor_url = "192.168.172.100:9090"
def url = "193.168.172.100"
def harbor_project_name = "chitang-mail"
node {
    stage('Preparation') {
            checkout scmGit(branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: 'd0713e0a-bca2-4ce2-b74f-3abe0fe00f85', url: 'http://192.168.172.100:8090/root/chitang-mail.git']])
            echo '拉取代码成功'
        }
    stage('maven build') {
        sh ''' /home/local/apache-maven-3.6.3/bin/mvn clean package -DskipTests '''
        sh "/home/local/apache-maven-3.6.3/bin/mvn dockerfile:build -DskipTests"
        echo '构建成功'
    }

    stage ('docker build') {
        def realName = "chitang-mail"
        def imageName = "${realName}:${Tag}"
        def currentProjectPort = "5429"
        echo "生成镜像：${realName}"
        echo "镜像名称：${imageName}"
        sh "docker tag ${imageName} ${harbor_url}/${harbor_project_name}/${imageName}"
        echo "给镜像：${imageName}打标签"
        withCredentials([usernamePassword(credentialsId: 'aa6aff56-43a4-4063-a0ab-1ca381b7ac25', passwordVariable: 'password', usernameVariable: 'username')]) {
            echo "开始登录harbor"
            sh "docker login -u ${username} -p ${password} http://${harbor_url}"
            echo "登录harbor成功"
            sh "docker push ${harbor_url}/${harbor_project_name}/chitang-mail:${Tag}"
            echo "上传镜像：${imageName}成功"
        }
        sh "docker rmi -f ${imageName}"
        echo "删除镜像成功"
        sh "docker rmi -f ${harbor_url}/${harbor_project_name}/${imageName}"
        echo "删除harbor镜像：${imageName}成功"
        echo "开始部署项目" 
        sshPublisher(publishers: [sshPublisherDesc(configName: '192.168.172.100', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: "/home/local/deploy.sh > deploy.log ${harbor_url} ${harbor_project_name} ${realName} ${tag} ${currentProjectPort}", execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
    }
}
```

------

## Jenkinsfile文件

```nginx
//gitlab的凭证
def git_auth = "2b5e41bd-2b9b-400d-9500-910f90f7b35c"

//构建版本的名称
def tag = "latest"
def frontend_tag = "1.0.0"
def frontend_project_name = "jcwy-auth-web"
def port = "19100"

//Harbor私服地址
def harbor_url = "192.168.5.70:9090"
def url = "192.168.5.70"

//Harbor的后端项目名称
def harbor_backend_project_name = "jcwy-auth"
//Harbor的前端项目名称
def harbor_frontend_project_name = "jcwy-auth-web"

// harbor用户凭证（全局凭证管理那里配置）
def harbor_auth = "9037160f-6b94-4d6c-b955-921a90a1f884"
node {
        stage('拉取代码') {
            checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: 'http://8.142.127.190:8010/wangjunsheng/jcwy-auth.git']]])
        }
        def buildType = "${build_type}"
        echo "选择的构建类型： ${buildType}"
        if (buildType == "") {
            echo "请选择构建类型==>>前端OR后端"
            return
		}
        else {
		if (buildType == "前端") {
            echo "开始构建前端......"
            def frontendImageName = "${frontend_project_name}:${frontend_tag}"

            stage('构建前端') {
                nodejs('nodejs16.0.0') {
                    sh '''
                        cd ./ruoyi-ui
                        pwd
                        echo "安装插件"
                        npm install --legacy-peer-deps --registry=https://registry.npmmirror.com
                        echo "打包"
                        npm run build:prod
                    '''
                }
                echo "构建镜像： ${frontendImageName}"
                sh "docker build -f ./ruoyi-ui/Dockerfile -t ${frontendImageName} ."
                echo "给镜像：${frontendImageName}打标签"
                sh "docker tag ${frontendImageName} ${harbor_url}/${harbor_frontend_project_name}/${frontendImageName}"
            }
            stage('部署前端') {
                withCredentials([usernamePassword(credentialsId: "${harbor_auth}", passwordVariable: 'password', usernameVariable: 'username')]) {
                //登录
                echo "开始登录"
                sh "docker login -u ${username} -p ${password} ${harbor_url}"
                echo "登录harbor成功"
                //上传镜像
                sh "docker push ${harbor_url}/${harbor_frontend_project_name}/${frontendImageName}"
                echo "上传镜像:${frontendImageName}成功"
                }
                //删除本地镜像 （根据镜像名称删除镜像和打标签的镜像，因为原本的镜像和打包后镜像是同一个镜像id,无法通过镜像id删除镜像）
                echo "删除本地镜像"
                sh "docker rmi -f ${frontendImageName}"
                echo "删除harbor镜像"
                sh "docker rmi -f ${harbor_url}/${harbor_frontend_project_name}/${frontendImageName}"
                // 部署项目
                echo "开始部署项目"
                sshPublisher(publishers: [sshPublisherDesc(configName: '192.168.5.70', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: "/home/jcwy-auth/deploy-web.sh ${harbor_url} ${harbor_frontend_project_name} ${frontend_project_name} ${frontend_tag} ${port} > deploy.log 2>&1", execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
		}
        else if (buildType == "后端") {
            //把选择的项目信息转为数组
            def projectName = "${module_name}"

            echo "${projectName}"
            echo "开始构建后端......"
            stage('编译构建') {
                //编译并安装公共工程
                sh "mvn clean package -DskipTests=true"
                //取出每个项目的名称和端口
                //项目名称
                def realName = projectName.split('@')[0]
                //项目启动端口
                def currentProjectPort = projectName.split('@')[1]
                def imageName = "${realName}:${tag}"
                // 编译打包选中微服务项目
                //echo "编译打包选中微服务项目:${realName}"
                //sh "mvn -f ${realName} clean package"

                // 生成镜像
                echo "生成镜像:${realName}"
                sh "mvn -f ${realName} dockerfile:build"

                //def imageName = "${realName}:${tag}"
                echo "镜像名称:${imageName}"
                //给镜像打标签
                sh "docker tag ${imageName} ${harbor_url}/${harbor_backend_project_name}/${imageName}"
                //sh "docker build -t ${imageName} /var/jenkins_home/workspace/aiji/${realName} -f Dockerfile"
                echo "给镜像:${imageName}打标签"

                withCredentials([usernamePassword(credentialsId: "${harbor_auth}", passwordVariable: 'password', usernameVariable: 'username')]) {
                    //登录
                    echo "开始登录"
                    sh "docker login -u ${username} -p ${password} ${harbor_url}"
                    echo "登录harbor成功"
                    //上传镜像
                    sh "docker push ${harbor_url}/${harbor_backend_project_name}/${imageName}"
                    echo "上传镜像:${imageName}成功"
                }

                //删除本地镜像 （根据镜像名称删除镜像和打标签的镜像，因为原本的镜像和打包后镜像是同一个镜像id,无法通过镜像id删除镜像）
                echo "删除本地镜像"
                sh "docker rmi -f ${imageName}"
                echo "删除harbor镜像"
                sh "docker rmi -f ${harbor_url}/${harbor_backend_project_name}/${imageName}"
                // 部署项目
                echo "开始部署项目"
                echo "currentProjectPort: $currentProjectPort"
                sshPublisher(publishers: [sshPublisherDesc(configName: '192.168.5.70', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: "/home/jcwy-auth/deploy.sh ${harbor_url} ${harbor_backend_project_name} ${realName} ${tag} ${currentProjectPort} 1 > deploy.log", execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])

                    }
				}
        else {
            echo "构建类型数据异常！！！"
            return
        }
		}
}
```

注意格式

![img](media/img-2.png)

------

### deploy.sh脚本

将deploy.sh脚本传到服务器上，并执行 chmod -R 777 deploy.sh

```shell
#! /bin/sh
#接收外部参数
harbor_url=$1
harbor_project_name=$2
project_name=$3
tag=$4
port=$5
imageName=$harbor_url/$harbor_project_name/$project_name:$tag
echo "$imageName"
#查询容器是否存在，存在则删除
# docker ps -a 查询所有的容器（无论启动还是停止）
# grep -w 根据一个单词来全匹配查询
# awk '{print $1}' 对于获取来的每行数据，按空格分隔，取第1列的值，$2表示拿第二列的值
containerId=`docker ps -a | grep -w ${project_name}:${tag} | awk '{print $1}'`
# containerId=`docker ps -aux |grep tvp | grep -v grep | awk ‘{print $1}’ | xargs kill -9`
echo "containerId: $containerId"
echo "开始删除容器"
if [ "$containerId" != "" ] ; then
    #停掉容器
    docker stop $containerId
    #删除容器
    docker rm $containerId
    echo "成功删除容器"
fi
#查询镜像是否存在，存在则删除
imageId=`docker images | grep -w $project_name | awk '{print $3}'`
echo "imageId: $imageId"
echo "开始删除镜像"
if [ "$imageId" != "" ] ; then
    cId=`docker ps -a | grep -w ${imageId} | awk '{print $1}'`
    echo "cId: $cId"
    if [ "$cId" != "" ] ; then
      #停掉容器
      docker stop $cId
      #删除容器
      docker rm $cId
      echo "成功删除镜像容器"
    fi
    #删除镜像
    docker rmi -f $imageId
    echo "成功删除镜像"

fi
# 登录Harbor私服（自己手动登录harbor的话记得加上harbor的端口号，例如docker login -u admin -p Harbor12345 127.0.0.1:85）
docker login -u admin -p Harbor12345 $harbor_url
# 下载镜像
docker pull $imageName
# 启动容器
if [ "$port" == "5429" ] ; then
  docker run -di -p $port:$port -v /home/chitang/$project_name/logs/:/chitang/$project_name --name $project_name $imageName
else 
  docker run -di -p $port:$port -v /home/aiji/$project_name/logs/:/aiji/$project_name --name $project_name $imageName
fi
echo "容器启动成功"
```

------

```shell
//gitlab的凭证

// 192.168.5.55
def git_auth = "a60348d5-c587-4b24-aa41-b697739b9085"

//构建版本的名称
def tag = "latest"

//Harbor私服地址
def harbor_url = "127.0.0.1:85"

def url = "47.101.213.231"

//Harbor的项目名称
def harbor_project_name = "aiji"

// harbor用户凭证（全局凭证管理那里配置）

// 192.168.5.55
def harbor_auth = "adc98f5c-1d3a-4c66-89ea-8302b98eb8b7"

node {
    //把选择的项目信息转为数组
        def selectedProjects = "${project_name}".split(',')
        echo "${selectedProjects}"

        stage('拉取代码') {
            checkout([$class: 'GitSCM', branches: [[name: '*/${branch}']], extensions: [], userRemoteConfigs: [[credentialsId: "${git_auth}", url: 'http://8.142.127.190:8010/auto-parts/aiji.git']]])
        }

        stage('编译构建') {
            //编译并安装公共工程
            sh "mvn -f common clean install"

            for(int i=0;i selectedProjects.size();i++){
                //取出每个项目的名称和端口
                def currentProject = selectedProjects[i];
                //项目名称
                def realName = currentProject.split('@')[0]
                //项目启动端口
                def currentProjectPort = currentProject.split('@')[1]

                // 编译打包选中微服务项目
                echo "编译打包选中微服务项目:${realName}"
                sh "mvn -f ${realName} clean package"

                // 生成镜像
                echo "生成镜像:${realName}"
                sh "mvn -f ${realName} dockerfile:build"

                def imageName = "${realName}:${tag}"
                echo "镜像名称:${imageName}"
                //给镜像打标签
                sh "docker tag ${imageName} ${harbor_url}/${harbor_project_name}/${imageName}"
//                 sh "docker build -t ${imageName} /var/jenkins_home/workspace/aiji/${realName} -f Dockerfile"
                echo "给镜像:${imageName}打标签"


                withCredentials([usernamePassword(credentialsId: "${harbor_auth}", passwordVariable: 'password', usernameVariable: 'username')]) {
                    //登录
                    echo "开始登录"
                    sh "docker login -u ${username} -p ${password} ${harbor_url}"
                    echo "登录harbor成功"
                    //上传镜像
                    sh "docker push ${harbor_url}/${harbor_project_name}/${imageName}"
                    echo "上传镜像:${imageName}成功"
                }

                //删除本地镜像 （根据镜像名称删除镜像和打标签的镜像，因为原本的镜像和打包后镜像是同一个镜像id,无法通过镜像id删除镜像）
                echo "删除本地镜像"
                sh "docker rmi -f ${imageName}"
                echo "删除harbor镜像"
                sh "docker rmi -f ${harbor_url}/${harbor_project_name}/${imageName}"
                // 部署项目
                echo "开始部署项目"
                sshPublisher(publishers: [sshPublisherDesc(configName: '47.101.213.231', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: "/home/aiji/deploy.sh ${harbor_url} ${harbor_project_name} ${realName} ${tag} ${currentProjectPort}", execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
    }
}
```

------

## 多选框插件

Extended Choice Parameter

![img](media/img-5.png)

![img](media/img-3.png)

![img](media/img-3.png)