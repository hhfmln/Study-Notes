### git

##### 1.设置用户名邮箱

```
##### 
git config --global user.name '你在GitHub注册的用户名'
git config --global user.email '你在GitHub注册的邮箱'
```

##### 2.使用以下命令，查看设置的用户名和邮箱

```
git config user.name
git config user.email
```

##### 3.生成ssh公钥

```
ssh-keygen -t rsa -C '你在GitHub注册的邮箱'
```

##### 4.GitHub配置SSH公钥

```
在此路径下用记事本打开文件（id_rsa.pub）并全选复制里面的内容C:\Users\你的用户名\.ssh
在GitHub创建公钥，粘贴进去。
```

##### 5.上传项目

1.初始化仓库

```
git init
```

2.上传文件，创建readme.md

```
touch README.md
```

3.依次使用以下命令将文件夹的文件全部添加到暂存区并提交

```
git add .
git commit -m 'first commit'
```

6.提交完成后，与远程仓库连接

```
git remote add origin github的链接
```

7.最后一步将本地仓库的内容推送到远程仓库

```
git push -u origin master
```

