## gitbub 相关

### 使用 ssh 与本地 pc 建立连接

1 terminal 中设置使用 Git 时的姓名和邮箱地址

    git config --global user.name "lee527"
    git config --global user.email "lee94527@outlook.com"

2 生成 ssh Key

     ssh-keygen -t rsa -C "your_email@example.com"
     ...
     (一路回车，不要设置密码 ，否则每次与github的push操作需要输入密码)

3 查看公钥

    cat ~/.ssh/id_rsa.pub

4 将公钥添加至 github 个人设置左侧的 SSH and GPG keys 中，title 任意，key type 默认，将 ⬆ 上一步中输出的密钥 copy 到 key 中。

5 与 GitHub 进行认证和 通信

    ssh -T git@github.com

6 terminal 成功返回

    Hi  [GitHub_username]! You've successfully authenticated, but GitHub does not
    provide shell access.

### 基本 git 使用

1.  在需要的某目录文件下初始化 git

        git init

2.  将需要记录的文件提交至暂存区

        git add filename
        或
        git add ./*

3.  正式提交保存至历史记录

        git commit -m "message"

4.  反悔了，想要退回到某次提交

        git reset --hard a1f1a49679e048448a59ad7dc8b499416882a473
        (通过 git log 查看提交记录，每条记录会有一个唯一的长 id，如上例所示)

5.  推送至远程仓库 Github

        git remote add origin git@github.com:lee4527/frontEnd.git
        //origin 为远程仓库别名  后面http 为远程仓库地址，这个命令只用执行一次，以后直接push就行，此地址换成自己对应 Github账号下对应的仓库地址，这里我采用的是 ssh 验证的方式
        git push -u origin master
        //上面命令将本地的master分支推送到origin主机，同时指定origin为默认主机，后面就可以不加任何参数使用git push了。不带任何参数的git push，默认只推送当前分支，这叫做simple方式。
