```groovy
pipeline {

    environment {
        project_output_folder = "dist"
        project_name = "4"
        project_path = "/usr/local/nginx/html/v51"
        project_git_url = "http://10.5.0.42:8880/phoebus/solax-cloud.git"
        git_credential = "e9b16834-200b-4b8a-bed7-a4c91209ff5d"
        publish_host = "10.5.0.41"
        publish_host_name = "solax01"
        publish_host_ssh_username = "root"
        publish_host_ssh_password = "al20160826@solax"
    }

    agent any

    stages {

        stage('Checkout') {
            steps {
                dir("${project_name}"){
                    git credentialsId: "${git_credential}", branch: "master", url: "${project_git_url}"
                }
            }
        }

        stage("Unifying Distribution Folder Name"){
            steps{
                sh '''
                    cd ${project_name}
                    sed \'/outputDir/d\' vue.config.js > vue.config
                    mv vue.config vue.config.js
                '''
            }
        }

        stage('Build') {
            steps{
                nodejs('node16') {
                sh '''
                    cd ${project_name}
                    ls -l
                    npm cache clear --force
                    npm install --legacy-peer-deps --registry=http://mirrors.cloud.tencent.com/npm/
                    npm run build
                '''
                }
            }
        }

        stage("Compress"){
            steps{
                sh '''
                    cd ${project_name}
                    mv ${project_output_folder} ${project_name}
                    tar -cf ${project_name}.tar ./${project_name}
                    mv ${project_name}.tar ..
                '''
            }
        }

        stage("Publish"){
            steps{
                script{
                    def remote = [:]
                    remote.name = "${publish_host_name}"
                    remote.host = "${publish_host}"
                    remote.user = "${publish_host_ssh_username}"
                    remote.password = "${publish_host_ssh_password}"
                    remote.allowAnyHosts = true
                    sshCommand remote: remote, command: "if [ -d \"${project_path}/${project_name}\" ]; then \n mv ${project_path}/${project_name} ${project_path}/backup/${project_name}_\$(date \"+%Y%m%d%H%M\"); \n fi"
                    sshPut remote: remote, from: "${project_name}.tar", into: "${project_path}"
                    sshCommand remote: remote, command: "tar -xf ${project_path}/${project_name}.tar --directory=${project_path}"
                    sshCommand remote: remote, command: "rm -f ${project_path}/${project_name}.tar"
                }
            }
        }
    }
}
```