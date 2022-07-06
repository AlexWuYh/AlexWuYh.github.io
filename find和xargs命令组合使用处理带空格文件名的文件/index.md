# find和xargs命令组合使用处理带空格文件名的文件


当用`find`命令查找文件然后用`xargs`来批量处理文件时，当文件名中包含空格字符时，就会导致处理失败，因为xargs会认为空格前后分别是两个不同的文件。如下图：
![](https://cdn.jsdelivr.net/gh/alexwuyh/pic-host@master/photo/20220706110451.png)


我们查看`find`命令帮助文档可以发现，它有一个专门针对该情况并配合`xargs`命令的参数：`-print0`

```bash
-print0
              True; print the full file name on the standard output, followed by a null character (instead of the newline character that -print uses).  This allows file names that con‐
              tain newlines or other types of white space to be correctly interpreted by programs that process the find output.  This option corresponds to the -0 option of xargs.
```
与`find`默认的`-print`参数相比，它输出的序列不是以空格分隔，而是以`null`字符分隔。而`xargs`也有一个参数`-0`，可以接受以`null`而`非空格`间隔的输入流。

所以，假如我们要找到当前目录下所有文件名以`1).jpg`结尾的文件并将它们全部删除掉时，就可以像下面这样操作：

```bash
find . -name "*1).jpg" -print0 | xargs -0 rm -f
```
![](https://cdn.jsdelivr.net/gh/alexwuyh/pic-host@master/photo/20220706110529.png)
