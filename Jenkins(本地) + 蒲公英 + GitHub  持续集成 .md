### Jenkins(本地) + 蒲公英 + GitHub  持续集成 

#### 一.Jenkins 本地安装
使用brew 进行安装  [自行Google](www.google.com)

`默认已经安装好brew了,下面开始操作了`

###### jenkins命令行安装
>  在终端输入命令行 `brew install jenkins`, jenkins 基于java环境,所以命令执行后,可能会看到
>

```
jenkins: Java 1.7+ is required to install this formula.
JavaRequirement unsatisfied!

You can install with Homebrew-Cask:
  brew cask install java
```
> 然后根据提示 在终端中输入 `brew cask install java` ,安装过程需要输入Mac密码
> 安装java环境完成之后,提示 `🍺  java was successfully installed!`
> 
> 安装java成功之后,重新执行 `brew install jenkins`, 即可.等待完成
> 结束以后,默认端口8080,可以通过在终端输入, `open/usr/local/Cellar/jenkins` , 命令查看是否安装成功
> 
> 运行安装包,安装jenkins 客户端
> 
> 
> **/Users/tingbo/.jenkins/secrets/initialAdminPassword**
> **54609ac349dd4a9b81dbe9490601c6be**
> 
> 
> 
> 安装完成后打开终端输入命令行：`cd /usr/local/Cellar/jenkins/版本号/libexec` , 接着再输入命令行： ` java -jar ./jenkins.war`,    此时终端就会打印出一些日志，不用理会终端，也不要关闭
> 查看终端输入内容 `Jenkins is fully up and running` 看到这句话表示,安装好了,下面进入配置阶段


###### jenkins配置
> 打开网址 [localhost:8080](http://localhost:8080/login?from=%2F). 第一次打开,会进入到初次登录页面 ![](Jenkins注册页面.png)
> 按照提示进入到路径文件夹下获取密码即可,也可以从刚刚终端输出位置找到默认密码即可 `/Users/用户名/.jenkins/secrets/initialAdminPassword` 
> ![](/Users/tingbo/Documents/随手笔记/Jenkins终端密码提示.png)
> 
> 后续步骤入下
> ![](Jenkins配置选择页.png)
> ![](Jenkins安装进度页.png)
> 安装过程保证网络良好,如果卡着不动了,参考下面一个朋友的解决办法.
> 
> 就在上图所示的这一步下载的时候，基本上最后肯定有几个插件没有下载好，然后就停在这里不动了，不管怎么都不动，然后关机重启,重新进入.wars所在的文件路径，在终端执行`java -jar jenkins.war --httpPort=8888`命令才能在输入`http://localhost:8888`后能登录Jenkins登录界面，才跳到下图所示的管理员用户注册页面
> 
> ![](/Users/tingbo/Documents/随手笔记/Jenkins注册页面.png)
> 然后就是常规注册,不表!
> 

