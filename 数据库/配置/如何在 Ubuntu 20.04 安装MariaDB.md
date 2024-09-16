MariaDB是一个开放源代码，多线程关系型数据库管理系统，是 MySQL 的向后兼容替代品

-   [![myfreax](data:image/webp;base64,UklGRmoEAABXRUJQVlA4WAoAAAAQAAAAHwAAHwAAQUxQSNsBAAABkHZb25nH6p/athl0bNu2bdu2bdu2bdueqe2+TnJ9eJI0ExETQFSW2PhXMH6YHki0bvJeAgDLZV6jij+geM1fE7t9UBZGa8L8U4Hrzlo00Kn5Gfa/4st5u1gXi/2npuj1m9tbu3gWw+GUGlnTXnd1VtuKhdzq6kjLNDlLkUJ2FRVWIU17dZqVTmVM3KTw0ldFm3cG0fz68Lc7It75dDTKPYpQCn0H+kyZtUBhC69bcjjorNBAJ/O4zh8At3zLPxQAQcCXYIWmBpm35wFAXGwbuBMFM/rM62SjUDFTJt9AIacNWY3MCkTtVBGABOX3A/8BS6xVWC0/t/ix+FynJBVJQFp9FcTHxXo34gUl+VflVBBSJw0aXnRXwT6ClgX1lex2Q1NxgFLId23MnZWCv2rzI1bJZgMl5Esy5myJKppgK2cfWXdCCgBTspwhXqR+b5zbv4aPDXGqfCVfJ0LW+PgXVIuvOoQ6h/IDtyxJgCVeB1wq1+3LVwHmeB0swu3Tv4ZyfKhLOMuybY8uqDc7J7d3TFzt5t9M2+uNPrF0RdUS9Uqy4S7Exi0wIppjGL7fiBIcx7DdxpZhGJ5lmNjwADcbQts6ufsGhYVHxLIsx8VGx8ZEhQX5ejjZEkIIAFZQOCBoAgAAUA4AnQEqIAAgAD6ROJZHpaMiITAIALASCWwAwRVBOo/o1MBwAOYB/ZrAA3inyZqWk9Jx+v8B4gbV3eCOEfy3/Jd4FAq8wNG96Df+L5PtQb9U/9L+aned9FX9gFt39jHS/ayon8wEnc1E+6Ehu+XloXmMQZCzaI1+FZgAAP7/OlArOJg+m8aTkofO99+tBmzz/5RUt09fHtgL+FwDYhLG1U4Ne2n8l8qpof1itu7GQtHysDhk/Qe2uK+XzFk8CSpdJ4Y/4/qqLO13gPCm7zONXKsC5zTd2qOfuuJCvbXf9V6Ge/MMoOi1aubP/zRL2vhCbRjG6l88794/ofph/LNew11j6MHyK6dHD9551oRwdSeF5fQsSGevD7/0jZhfe4ttlU0G5JHgux1q8PCvonch2p+H9TWcTxurlM0LEHh03TeWI+h0qVVWqEvqmtX/+MWur/8WYMRrPvTH46PEaTyOYZ7+mLlmQzPyXe0s3bVt+8FOj9fh/J3X/31qcTRr3Tf1g+udh8UH/p/x9EOIWnn9HVPlMzsJG0Sq/u+g/mt29v5Z4P0Choo61qByRp7qCYjx0yerQkEfOPGNhnQEoWOiQCYkCDlrLzTpVrGZ1fl8ArxhQrjOKsOBWM2InypRHm3mXEDE6bMd739Qt4dfhb7I4YaqsJFIajg8fSlH4yw2vqfM3OrrM0qvZnK0V1qE+WGGZ/aOK5BlgJIXX8LoSs+amKTx9PIJB595ubhsSIyUocIx3v1UJrsgQEGm2XqRGs4WInQtXeo8mO4w5eMVrazXHbmeN+am4tj2W2M/ux60h0/2UZCVkFgAAA==)](https://www.myfreax.com/how-to-install-mariadb-on-ubuntu-20-04/ "Go to the profile of")

Updated At 11 Jan 2023 4 min read

By [myfreax](https://www.myfreax.com/author/myfreax/)

![如何在 Ubuntu 20.04 安装MariaDB](media/如何在_Ubuntu_20.04_安装MariaDB.webp)

如何在 Ubuntu 20.04 安装MariaDB

MariaDB是一个开放源代码，多线程关系型数据库管理系统，是 MySQL 的向后兼容替代品。它由 [MariaDB Foundation](https://mariadb.org/about/) 进行维护和开发。

在本教程中，将向您展示如何从 MariaDB 软件源在 Ubuntu 20.04 安装 MariaDB 最新版本。在撰写本文时，官方 MariaDB 软件源最新版本是10.9。

如果要安装 MySQL 而不是 MariaDB，请查看教程：如何在 [Ubuntu 20.04 安装MySQL教程](https://www.myfreax.com/how-to-install-mysql-on-ubuntu-20-04/)。

## 安装 MariaDB

在继续下一步之前，您应该访问 [MariaDB 仓库](https://downloads.mariadb.org/mariadb/repositories/)页面，检查是否有可用的新版本。要在 Ubuntu 20.04 安装 MariaDB 10.9，请执行以下步骤。

由于 MySQL 与 MariaDB 在动态库依赖存在冲突，因此在安装  MariaDB 10.9 之前请卸载 MySQL，解决动态库依赖存在冲突的问题。

然后运行命令 [curl命令](https://www.myfreax.com/curl-command-examples/)下载并导入MariaDB GPG密钥到 Ubuntu 20.04 。然后运行命令 sh 命令导入 MariaDB 软件源。

如果你需要安装其它版本的MariaDB 数据库，只需要改变 URL 版本号 10.9 为你需要的版本号即可。

```shell
sudo apt install software-properties-common
sudo apt purge mysql-server mysql-common
sudo apt autoremove

sudo curl -o /etc/apt/trusted.gpg.d/mariadb_release_signing_key.asc 'https://mariadb.org/mariadb_release_signing_key.asc'


sudo sh -c "echo 'deb https://mirrors.aliyun.com/mariadb/repo/10.9/ubuntu focal main' >>/etc/apt/sources.list"
```

现在已在 Ubuntu 20.04 添加 MariaDB 软件源，接下来安装MariaDB，运行`sudo apt update`命令更新软件索引。

然后运行命令`sudo apt install mariadb-server`，在安装过程可能会提示你输入密码，我们建议你留空，我们将在下一节中创建专用管理用户。

安装完成后，MariaDB将作为[systemd服务](https://www.myfreax.com/use-systemctl-management-linux-service/)自动启动，可以运行命令`sudo systemctl status mariadb`[查看 MariaDB 服务的状态](https://www.myfreax.com/use-systemctl-management-linux-service/)。

```bash
sudo apt update
sudo apt install mariadb-server
sudo systemctl status mariadb
```

```
● mariadb.service - MariaDB 10.3.8 database server
Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
Drop-In: /etc/systemd/system/mariadb.service.d
        └─migrated-from-my.cnf-settings.conf
Active: active (running) since Sun 2018-07-29 19:36:30 UTC; 56s ago
    Docs: man:mysqld(8)
        https://mariadb.com/kb/en/library/systemd/
Main PID: 16417 (mysqld)
Status: "Taking your SQL requests now..."
    Tasks: 31 (limit: 507)
CGroup: /system.slice/mariadb.service
        └─16417 /usr/sbin/mysqld
```

## MariaDB root用户密码

当 MariaDB 安装完成后，你可能会想运行命令`mysql -u root -p`登录到 MariaDB 服务器。

如果你登录到 Ubuntu 的用户不是 root 用户你将不能访问 MariaDB 服务器。如果你尝试使用密码登录也将被拒绝连接，因为在安装的过程我们并没有设置密码。

你将会收到类似于这样的消息(28000): Access denied for user 'root'@'localhost' (using password: YES)或者ERROR 1045 (28000): Access denied for user 'root'@'localhost'。

这意味着您无法通过提供密码以 root 用户连接到 MariaDB 服务器。但你可以通过命令 sudo mysql 连接到 MariaDB 服务器。

```shell
sudo mysql
```

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
mysql>
```

如果你要使用外部程序连接到 MariaDB，例如 phpMyAdmin。以 root 用户连接到 MariaDB 服务器。

则需要创建一个用于管理 MariaDB 数据库的用户，该用户可以访问所有数据库。运行SQL 语句 GRANT ALL ....。

当创建管理用户后，就可以通过新的管理用户使用密码的方式登录，可以在本地计算机运行命令`mysql -u admin -p`进行测试。

```sql
GRANT ALL ON *.* TO 'admin'@'localhost' IDENTIFIED BY '你的密码' WITH GRANT OPTION;

FLUSH PRIVILEGES;
```

如果你需要配置 MariaDB 用户的远程访问，可阅读我们的教程：[如何允许MySQL数据库服务器的远程连接](https://www.myfreax.com/mysql-remote-access/)**。**

[

如何允许MySQL数据库服务器的远程连接 | myfreax

默认情况下，MySQL服务器仅监听来自本地主机的连接，这意味着它只能由运行在同一主机上的应用程序访问

![](media/favicon-192.png)myfreaxmyfreax

![](media/mysql-remote-access.webp)

](https://www.myfreax.com/mysql-remote-access/)

## 结论

至此，你已经了解如何从 MariaDB 软件源在 Ubuntu 20.04 安装 MariaDB 最新版本。如有疑问，请在下面发表评论。