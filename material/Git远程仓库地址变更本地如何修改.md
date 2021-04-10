> 以下均以项目git_test为例：
>
> 老地址：http://gitlab.james.com/james/git_test.git
>
> 新地址：http://gitlab.james.com/janna/git_test.git
>
> 远程仓库名称：origin

#### 方法：

1. 通过命令直接修改远程地址
   1. 进入git_test根目录
   2. git remote 查看所有远程仓库，git remote xxx 查看指定远程仓库地址
   3. git remote set-url origin http://gitlab.james.com/janna/git_test.git
2. 通过命令先删除再添加远程仓库
   1. 进入git_test根目录
   2. git remote 查看所有远程仓库，git remote xxx 查看指定远程仓库地址
   3. git remote rm origin
   4. git remote add origin http://gitlab.james.com/janna/git_test.git
3. 直接修改配置文件
   1. 进入git_test/.git
   2. vim config
   3. 修改 [remote "origin"] 下面的URL
4. 通过第三方git 客户端修改
   1. 以 sourceTree 为例，点击 仓库 -> 仓库配置 -> 远程仓库，即可管理此项目中配置的所有远程仓库
   2. sourceTree界面最下方还可以点击编辑配置文件，同样可以完成方法3
