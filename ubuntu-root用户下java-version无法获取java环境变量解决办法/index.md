# Ubuntu root用户下java -version无法获取java环境变量解决办法

## 问题现象

​	按照网上配置java环境变量的方法在/etc/profile文件中添加如下内容，配置之后，如果重启系统后切换到root用户无法获取已配置的java环境变量，在普通用户下可以获取到

```shell
export JAVA_HOME=/usr/local/java/jdk1.8.0_65

export JRE_HOME=${JAVA_HOME}/jre

export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH

export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```
## 解决办法
<!--more-->
```shell
sudo ln -s /你的jdk路径/bin/jar /bin/jar 

sudo ln -s /你的jdk路径/bin/java /bin/java 

sudo ln -s /你的jdk路径/bin/javac /bin/javac 

sudo ln -s /你的jdk路径/bin/javah /bin/javah 

sudo ln -s /你的jdk路径/bin/javadoc /bin/javadoc
```

