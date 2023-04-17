#还没有复习 

`持续部署`，`持续集成`，`持续交付`



# 教程

这里有详细的教程

https://www.cnblogs.com/rmxd/p/11609983.html

以下是需要注意的地方

- 需要提前安装git，maven（修改源，指定本地库），jdk
- 云服务器生成ssh的rsa公私钥后要设置免密登录（否则在Jenkins测试连接云服务器时会报Auth错二连接不上）
- 在github上设置好webhook后，去Jenkins那设置去掉防跨域访问（否则github的钩子程序无法向Jenkins的接口发送请求，会报403）



# 下载安装，进入界面

1. 去[华为镜像站](https://mirrors.huaweicloud.com/jenkins/)下载`jenkins`的war包
2. 将war包放在`tomcat`的webapp目录下
3. 启动`tomcat`
4. 访问http://39.107.101.13:8080/jenkins/
5. ~~登录~~



# 以构建SpringBoot项目为目标构建任务



## 安装Git、Maven、JDK

以下为云服务器中上述软件的位置

`MAVEN_HOME=/opt/maven`

`JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64`



# 问题

- 创建一个项目后，可使用此项目同时构建多个任务

- 构建任务失败后（比如程序要用8080端口，结果8080已被占用），能不能kill掉占8080端口的程序后Jenkins的任务自动重新构建

- 关于脚本的编写，之前启动程序需要的`java -jar xxx`命令是通过以下脚本运行的

  ```shell
  ...
  nohup $JAVA_HOME/bin/java -classpath $classpath -XX:-UseGCOverheadLimit -Xms1024m -Xmx2048m -jar $d    ir/login.jar  > $dir/log/$(date +'%Y%m%d').log &
  ...
  ```

  不知道什么原因，不能用

  后修改为

  ```shell
  java -jar xxx.jar > /opt/jenkins_jars/login.log &
  ```

  结果运行脚本时，脚本输出不能创建"/opt/jenkins_jars/login.log"文件。后去掉日志输出后，能运行

  ```shell
  java -jar xxx.jar &
  ```

- 编写shell脚本时遇到的问题

  for循环不是用

  ```shell
  # 错误
  for xx in xxx{
  	...
  }
  ```

  而是用

  ```shell
  for xx in xxx
  do
  ...
  done
  ```

  