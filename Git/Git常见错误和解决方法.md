#### 一、git pull遇到错误：

> error: Your local changes to the following files would be overwritten by merge:

含义：对以下文件的本地更改将被合并覆盖

原因：在本地修改的时候没有提前拉取，导致在本地修改代码的时候远程代码已经有更新和提交

解决方法：

1. 如果想保留本地修改的代码，并把git服务器上面的代码pull到本地，即把本地修改的代码暂存

   ```sh
   git stash //暂存当前正在进行的工作
   git pull origin master // 拉取服务器代码
   git stash pop // 合并暂存的代码
   ```

2. 覆盖本地的代码，只保留服务器代码，则直接回退到上一个版本，再进行pull

   ```shell
   git reset --hard  //回滚到上一个版本
   git pull origin master
   ```

   
