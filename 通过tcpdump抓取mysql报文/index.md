# 通过tcpdump抓取mysql报文


```shell
tcpdump -s 65535 -xX -nn -q -tttt -i any -c 1000 port 3306
```
```shell
#只以ASIC显示内容
tcpdump -s 65535 -A -nn -q -tttt -i any -c 1000 port 3306
```
