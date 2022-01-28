# 让其他用户使用oh-my-zsh


如果使用`wt`用户安装配置了`oh-my-zsh`，其他用户想要使用相同的主题和配置，可以参考`https://askubuntu.com/questions/521469/oh-my-zsh-for-the-root-and-for-all-user`
这里介绍一种更简单的方法（亲测有效）：
比如让`root`用户使用和`wt`用户相同的配置：
<!--more-->
```shell
sudo ln -s $HOME/.oh-my-zsh           /root/.oh-my-zsh
sudo ln -s $HOME/.zshrc               /root/.zshrc
```
切换到`root`用户，命令`zsh`,即可看到`zsh`的主题和`wt`用户的一样了。如果提示
```shell
/root/.zshrc:119: command not found: pyenv
/root/.zshrc:120: command not found: pyenv
```
再创建`.pyenv`的软连接即可。
```shell
sudo ln -s $HOME/.pyenv    /root/.pyenv
```
这样做的缺点是`root`用户的所有配置都和`wt`用户的一致，不能个性化。修改一个，其他用户的也会变。
如果要个性化，可以用
```shell
sudo cp -r /home/wt/.oh-my-zsh    /root
sudo cp -r /home/wt/.zshrc    /root
```

