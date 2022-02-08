# MacOS上使用iterm2在跳板机上用lrzsz上传或下载文件


## 安装配置lrzsz/iterm2-zmodem
1. 安装:

```shell
brew install lrzsz
```
2. 打开终端下载支持脚本：

```shell
git clone git@github.com:mmastrac/iterm2-zmodem.git
```

3. 在终端下安装支持脚本：

```shell
cp iterm2-zmodem/iterm2-*.sh /usr/local/bin   &&   chmod +x /usr/local/bin/iterm2-*.sh
```

## 设置iterm2

![](https://qiniu.oneone.life/img/20220208162628.png)

![](https://qiniu.oneone.life/img/20220208162710.png)

```
Regular expression: rz waiting to receive.**B0100
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-send-zmodem.sh
Instant: checked
 
Regular expression: **B00000000000000
Action: Run Silent Coprocess
Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
Instant: checked
```


## 上传文件、下载文件

首先，通过跳板机登录到服务器。
下载文件：sz {要下载的文件名}
上传文件：rz
上传文件时，要要求你选择待上传的文件。
