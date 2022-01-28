# grep


用法: grep [选项]... PATTERN [FILE]...
在每个 FILE 或是标准输入中查找 PATTERN。
默认的 PATTERN 是一个基本正则表达式(缩写为 BRE)。
例如: `grep -i 'hello world' menu.h main.c`
 

- 正则表达式选择与解释:

    ```shell
      -E, --extended-regexp     PATTERN 是一个可扩展的正则表达式(缩写为 ERE)
      -F, --fixed-strings       PATTERN 是一组由断行符分隔的字符串。
      -G, --basic-regexp        PATTERN 是一个基本正则表达式(缩写为 BRE)
      -P, --perl-regexp         PATTERN 是一个 Perl 正则表达式
      -e, --regexp=PATTERN      用 PATTERN 来进行匹配操作
      -f, --file=FILE           从 FILE 中取得 PATTERN
      -i, --ignore-case         忽略大小写
      -w, --word-regexp         强制 PATTERN 仅完全匹配字词
      -x, --line-regexp         强制 PATTERN 仅完全匹配一行
      -z, --null-data           一个 0 字节的数据行，但不是空行
    ```
<!--more-->

- 杂项:

     ```shell
      -s, --no-messages         不显示错误信息
      -v, --invert-match        选中不匹配的行
      -V, --version             显示版本信息并退出
          --help                显示此帮助并退出
     ```

- 输出控制:
  ```shell
  -m, --max-count=NUM       NUM 次匹配后停止
  -b, --byte-offset         输出的同时打印字节偏移
  -n, --line-number         输出的同时打印行号
      --line-buffered       每行输出清空
  -H, --with-filename       为每一匹配项打印文件名
  -h, --no-filename         输出时不显示文件名前缀
      --label=LABEL         将LABEL 作为标准输入文件名前缀
  -o, --only-matching       只显示匹配PATTERN 部分的行
  -q, --quiet, --silent     不显示所有常规输出
      --binary-files=TYPE   设定二进制文件的TYPE 类型；
                            TYPE 可以是`binary', `text', 或`without-match'
  -a, --text                等同于 --binary-files=text
  -I                        等同于 --binary-files=without-match
  -d, --directories=ACTION  读取目录的方式；
                            ACTION 可以是`read', `recurse',或`skip'
  -D, --devices=ACTION      读取设备、先入先出队列、套接字的方式；
                            ACTION 可以是`read'或`skip'
  -r, --recursive           等同于--directories=recurse
  -R, --dereference-recursive       同上，但遍历所有符号链接
      --include=FILE_PATTERN  只查找匹配FILE_PATTERN 的文件
      --exclude=FILE_PATTERN  跳过匹配FILE_PATTERN 的文件和目录
      --exclude-from=FILE   跳过所有除FILE 以外的文件
      --exclude-dir=PATTERN  跳过所有匹配PATTERN 的目录。
  -L, --files-without-match  只打印不匹配FILEs 的文件名
  -l, --files-with-matches  只打印匹配FILES 的文件名
  -c, --count               只打印每个FILE 中的匹配行数目
  -T, --initial-tab         行首tabs 分隔（如有必要）
  -Z, --null                在FILE 文件最后打印空字符
  ```
  
- 文件控制:
  
  ```shell
  -B, --before-context=NUM  打印文本及其前面NUM 行
  -A, --after-context=NUM   打印文本及其后面NUM 行
  -C, --context=NUM         打印NUM 行输出文本
  -NUM                      等同于 --context=NUM
      --color[=WHEN],
      --colour[=WHEN]       使用标记高亮匹配字串；
                            WHEN 可以是`always', `never'或`auto'
  -U, --binary              不要清除行尾的CR 字符(MSDOS/Windows)
  -u, --unix-byte-offsets   忽略CR 字符，报告字节偏移
                          (MSDOS/Windows)
  ```
  
 
`egrep` 即`grep -E`。`fgrep` 即`grep -F`。
直接调用`egrep` 或是`fgrep` 均已被废弃。
若FILE 为 -，将读取标准输入。不带FILE，读取当前目录，除非命令行中指定了-r 选项。
如果少于两个FILE 参数，就要默认使用-h 参数。
如果有任意行被匹配，那退出状态为 0，否则为 1；
如果有错误产生，且未指定 -q 参数，那退出状态为 2。
 





