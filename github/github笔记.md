# github笔记

学习自 https://www.runoob.com/w3cnote/git-guide.html

## 本地电脑配置

1. 本地创建ssh key, your-email为github的注册邮箱。成功后会在当前文件夹下生成ssh文件夹。

```shell
ssh-keygen -t rsa -C "your-email"
```

2. 打开ssh文件夹。打开`id_rsa.pub`，复制里面的`key`。
   
   回到github上，进入 Account Settings（账户配置），左边选择SSH Keys，Add SSH Key,title随便填，粘贴在你电脑上生成的key。
   
   ![](C:\Users\zjw\AppData\Roaming\marktext\images\2022-04-11-15-23-27-github-account.jpg)

3. 验证是否成功
   
   ```shell
   ssh -T git@github.com
   ```

    如果是第一次的会提示是否continue，输入yes就会看到：You've successfully authenticated, but GitHub does not provide shell access 。这就表示已成功连上github。



这个时候就表示这一台电脑可以成功连接到自己的github

4. 设置自己的username和email
   
   ```shell
   $ git config --global user.name "your name"
   $ git config --global user.email "your_email@youremail.com"
   ```

## 创建与github相连的本次仓库

1. 在需要作为仓库的文件夹中打开git bash

2. 创建本地仓库
   
   ```shell
   git init
   ```

3. 连接到远程仓库。yourName和yourRepo表示你再github的用户名和刚才新建的仓库。加完之后进入.git，打开config，这里会多出一个remote "origin"内容，这就是刚才添加的远程地址，也可以直接修改config来配置远程地址。
   
   ```shell
   git remote add origin git@github.com:yourName/yourRepo.git
   ```

4. 本地仓库新建的内容，如果在开发工具中加入到暂存区。就不需要再单独添加，否则需要如下命令
   
   ```shell
   git add *
   or
   git add filename
   ```

5. 添加到head
   
   ```shell
   git commit -m "代码备注信息"
   ```

![](C:\Users\zjw\AppData\Roaming\marktext\images\2022-04-11-15-31-21-trees.png)

6. 发送到远程仓库,master为分支名字
   
   ```shell
   git push origin master
   ```
