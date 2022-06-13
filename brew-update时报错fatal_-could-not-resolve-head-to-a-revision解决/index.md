# brew update时报错“fatal_ Could not resolve HEAD to a revision”解决


## 报错信息
在安装brew并更改为国内下载源后，执行`brew update`时老是报错，信息如下：
![image.png](https://raw.githubusercontent.com/AlexWuYh/Pic-Host/master/photo/image%20(4).png)
## 解决办法
上网查了下，找到了解决办法，[参考链接](https://www.jianshu.com/p/b2de788c3c6d)
依次执行下面的命令/步骤：
```bash
brew update --verbose
```
![image.png](https://raw.githubusercontent.com/AlexWuYh/Pic-Host/master/photo/image%20(5).png)

依次进入执行后报错的目录中去，再分别执行命令:
```bash
git fetch --prune origin

git pull --rebase origin master
```
![image.png](https://raw.githubusercontent.com/AlexWuYh/Pic-Host/master/photo/image%20(6).png)
最后再次执行`brew update`就可以正常更新源了
![image.png](https://raw.githubusercontent.com/AlexWuYh/Pic-Host/master/photo/image%20(7).png)

