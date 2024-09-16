SH (Secure Shell) 是一种网络协议，用于远程登录和管理远程主机。常用的 SSH 命令有以下几种：

- ssh：连接到远程主机。格式为 ssh [user@]hostname [command]。
    
- scp：在本地和远程主机之间复制文件。格式为 scp [options] file [user@]host:destination。
    
- sftp：在本地和远程主机之间传输文件。格式为 sftp [user@]host。
    
- ssh-keygen：生成 SSH 密钥。格式为 ssh-keygen [options]。
    
- ssh-copy-id：将本地密钥复制到远程主机。格式为 ssh-copy-id [user@]host。
    

常用的参数有：

- -p：指定端口号
- -i：指定私钥文件
- -o：传递给 ssh 命令的附加选项
- -v：显示详细输出，可以使用 -vv 或 -vvv 来增加详细程度
- -C：启用压缩

例如：

```ruby
#连接远程主机
ssh user@host

#在本地和远程主机之间复制文件
scp -r -P port -i privatekey.pem file user@host:/remote/path

#在本地和远程主机之间传输文件
sftp -P port -i privatekey.pem user@host

#生成 SSH 密钥
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

#将本地密钥复制到远程主机
ssh-copy-id -i ~/.ssh/id_rsa.pub user@host
```