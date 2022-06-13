# brew update时报错“fatal_ Could not resolve HEAD to a revision”解决


## 报错信息
在安装brew并更改为国内下载源后，执行`brew update`时老是报错，信息如下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/770522/1654852075048-5743bc72-384b-4998-8881-c4fc906888f6.png#clientId=u57b4c630-a3bb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=113&id=u77a12e77&margin=%5Bobject%20Object%5D&name=image.png&originHeight=226&originWidth=1666&originalType=binary&ratio=1&rotation=0&showTitle=false&size=120971&status=done&style=none&taskId=ue2f7bb02-c339-427c-8668-89e58e22442&title=&width=833)
## 解决办法
上网查了下，找到了解决办法，[参考链接](https://www.jianshu.com/p/b2de788c3c6d)
依次执行下面的命令/步骤：
```bash
brew update --verbose
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/770522/1654852322102-91d1d82e-bf62-4ffb-9797-94ea6806a818.png#clientId=u57b4c630-a3bb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=439&id=ub71d3e03&margin=%5Bobject%20Object%5D&name=image.png&originHeight=878&originWidth=1698&originalType=binary&ratio=1&rotation=0&showTitle=false&size=515083&status=done&style=none&taskId=uc1f8b40b-ef00-431c-98b4-f811d0af8d6&title=&width=849)

依次进入执行后报错的目录中去，再分别执行命令:
```bash
git fetch --prune origin

git pull --rebase origin master
```
![image.png](https://cdn.nlark.com/yuque/0/2022/png/770522/1654852428022-90bb19e7-f4b4-4521-b085-f76e1a41e6ec.png#clientId=u57b4c630-a3bb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=346&id=u6c1874da&margin=%5Bobject%20Object%5D&name=image.png&originHeight=692&originWidth=1666&originalType=binary&ratio=1&rotation=0&showTitle=false&size=510734&status=done&style=none&taskId=u723bd273-5eec-442e-902b-8ad1b00dafa&title=&width=833)
最后再次执行`brew update`就可以正常更新源了
![image.png](https://cdn.nlark.com/yuque/0/2022/png/770522/1654852516041-a564e9ff-1b44-4620-9f8e-b1a18200b2fc.png#clientId=u57b4c630-a3bb-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=208&id=uc886e529&margin=%5Bobject%20Object%5D&name=image.png&originHeight=416&originWidth=1686&originalType=binary&ratio=1&rotation=0&showTitle=false&size=232776&status=done&style=none&taskId=ud36eed88-e3dc-437b-874c-78c235a39d3&title=&width=843)

