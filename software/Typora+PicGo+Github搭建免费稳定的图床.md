## PicGo](https://github.com/Molunerfinn/PicGo)

**一个用于快速上传图片并获取图片 URL 链接的工具**

PicGo 本体支持如下图床：

- `七牛图床` v1.0
- `腾讯云 COS v4\v5 版本` v1.1 & v1.5.0
- `又拍云` v1.2.0
- `GitHub` v1.5.0
- `SM.MS V2` v2.3.0-beta.0
- `阿里云 OSS` v1.6.0
- `Imgur` v1.6.0

### GIthub操作：

1. **新建Public Github仓库**
   1. 创建Repository
   2. 点击"New repository"按钮，仓库名字随意
2. **新生成一个Personal access tokens**
   1. 生成一个Token用于操作GitHub repository
      `Settings --> Developer Settings --> Personal access tokens`
   2. 勾选repo权限,填写描述然后点击"Generate token"按钮
   3. 生成Token

### PicGo配置：

![PicGo-Server配置](https://raw.githubusercontent.com/chnjames/cloudImg/main/image-20210224104844303.png)

1. 配置Github图床

   1. ![PicGo配置](https://raw.githubusercontent.com/chnjames/cloudImg/main/20210224103532.png)

      填写说明：

      - 第一行设定仓库名按照“账户名/仓库名的格式填写”，比如我的是：`chnjames/cloudImg`
      - 第二行分支名统一填写`main`
      - 第三行将之前的`Token`粘贴在这里
      - 第四行留空
      - 第五行自定义域名的作用是在上传图片后成功后，`PicGo`会将“自定义域名+上传的图片名”生成的访问链接，放到剪切板上，自定义域名需要按照这样去填写：`https://raw.githubusercontent.com/账户名/仓库名/main`

### [Typora](https://typora.io/)配置PicGo：

![Typora配置](https://raw.githubusercontent.com/chnjames/cloudImg/main/image-20210224105132603.png)

### 错误排查：

##### 错误一：Failed to fetch

![错误一](https://raw.githubusercontent.com/chnjames/cloudImg/main/aHR0cHM6Ly9naXRlZS5jb20vbGVvbkc3L2Jsb2dJbWFnZS9yYXcvbWFzdGVyL2ltZy8yMDIwMDMxODE0NDc0NC5wbmc)

**端口设置错误**造成

`解决方法`：打开picgo设置，点击设置Server选项，**将端口改为36677端口**，这是picgo推荐的默认端口号，然后保存，成功

##### 错误二：Failed to fetch：端口冲突，**picgo自动帮你把36677端口改为366771端口**，导致错误

`解决方法`：**先把picgo中的端口设置改回36677，然后退出所有picgo程序**，再使用typora上传功能（会自动启动picgo程序）

##### 错误三：{“success”,false}

![错误三](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vbGVvbkc3L2Jsb2dJbWFnZS9yYXcvbWFzdGVyL2ltZy8yMDIwMDMxODE0MjYyMy5wbmc?x-oss-process=image/format,png)

原因：**文件名冲突**

`解决办法`：打开picgo设置，将**【时间戳重命名】打开**



