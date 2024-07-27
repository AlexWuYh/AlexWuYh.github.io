# linux中curl调用登录接口然后用sed命令提取出token值



> 💡**背景：**<br />**因为客户现场特殊情况不能访问管理平台Web页面，且不能远程访问服务器和其中的所有虚拟机实例，只能在机房直接操作集群对应服务器。现场技支同事需要上传一个镜像文件到集群中的某一个微服务中，通常还可以通过Web页面进行操作，现在却不行。为了解决他们的这个述求，就想到直接从宿主机中调用对应服务的注册接口获取token，然后再直接调用上传的接口上传镜像文件。**


## 1. 把镜像文件上传到宿主机

- 把需要上传的镜像文件拷贝到集群中任一节点的宿主机系统中去(建议就NG节点，接口请求本身也就是先走NG)，目录随意记住就行。
## 2. 获取token
首先需要通过注册接口获取`token`用于后续接口调用的鉴权，但是`toeken`是一个很长的字符串，再加上只能操作纯命令行的Linux系统，所以就不能使用鼠标直接选择复制这种操作了。为了方便现场同事后续的操作，就只能把接口返回的`token`值直接写入到一个文件中或者一个变量函数，此处选择的是写文件。
### 2.1 方法一
直接将接口返回全部写入到一个指定文件，然后再编辑该文件，手动删掉无用的内容只保留`token`值的部分。好处是命令较短，手动敲入的时候出错的概率较低，缺点是操作较繁琐。
#### 调用登录接口获取token

- 在服务器上执行下方的命令获取token，并把token写入文件response.txt 
```bash
curl --location --request POST 'http://xx.xx.xx.xx/api/v2/login' --header 'Content-Type: application/json' --data-raw '{"userName": "xxxxx","password": "xxxxx"}' > response.txt
```

> 💡**参数说明：**
> - 使用`curl`命令直接调用登陆接口
> - 同时使用`>`把接口返回接口重定向写入到文件`response.txt`中去

#### 编辑`response.txt`文件，只保留`token`值部分的内容

- 使用`vim`命令编辑`response.txt`文件，只保留`token`字段的值那部分内容，其他内容删除掉(如下图)，然后`:wq`保存退出
   - 编辑前：
   ![](https://cdn.jsdelivr.net/gh/alexwuyh/pic-host@master/photo/202212291424420.png)
   - 编辑后：
   ![](https://cdn.jsdelivr.net/gh/alexwuyh/pic-host@master/photo/202212291425499.png)

## 2.2 方法二
将接口返回的值先进行处理，截取出`token`值部分的内容后再写入到一个指定的文件中去。好处是减少手动编辑文件删除内容截取token的步骤(毕竟接口返回内容还是比较多)，缺点是命令较长手动敲入时容易出错。
#### **调用登录接口获取token，并自动截取出token值部分内容**

- 执行下方的命令获取token，并把处理好的token值写入到文件中去
```bash
curl --location --request POST 'http://xx.xx.xx.xx/api/v2/login' --header 'Content-Type: application/json' --data-raw '{"userName": "xxxxx","password": "xxxxx"}' | sed 's/,/\n/g' | grep "token" | sed 's/:/\n/g' | sed '1d' | sed 's/"//g' > response.txt
```


> **💡参数说明：**
>
> - 使用`curl`命令直接调用登陆接口
> - 第一个`sed`是将` , `换成`\n`
> - 第二个`grep` 是将 `token` 关键字找出来，并单独列出来
> - 第三个 `sed`是将 `:` 换成 `\n`
> - 第四个`sed` 是删除第一行
> - 第五个`sed`是将 `"` 用空字符替换掉, 最后的`g`的参数表示替换所有符合的引号
> - 第六个`>`是将结果重定向写入到文件`response.txt`中


## 3. 调用上传接口，上传镜像文件

- 读取之前写入`token`的文件，并使用`xargs`命令传入`curl`的接口请求参数中去（当然还有其他的方法实现传参，此处就不做介绍了，一切以现场同事手动录入方便为先）
```bash
cat response.txt | xargs -I {} curl -v  --location --request POST 'http://xx.xx.xx.xx/api/v2/image' --header 'Authorization: Bearer {}' --form 'file=@"/home/app/xxxxxx.tar.gz"'
```

> **💡参数说明：**
>
> - `xargs`命令后面那个参数是大写的英文字母`i`
> - 把镜像包的路径替换为现场的真实路径（`file=@`后面那部分）


- **当**接口返回状态为`HTTP 100 Continue`，这表示镜像文件正在上传中，耐心等待，直到文件上传完成即可

    ![](https://cdn.jsdelivr.net/gh/alexwuyh/pic-host@master/photo/202211161059420.png)


---

我的博客即将同步至腾讯云开发者社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite_code=24be6fvhrt0ks

