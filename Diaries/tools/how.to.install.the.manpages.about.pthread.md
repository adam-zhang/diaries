# ubuntu下安装pthread的manpages（man 手册）

    由于学习多线程编程，所以用到pthread，但是man的时候却发现没有pthread函数库的手册页，然后安装
```
$sudo apt-get install glibc-doc
```
    安装以后，发现还是有很多函数不全，只有一小部分pthread的函数，使用man -k pthread或apropos pthread可以查找到当前manpages中关于pthread的手册。安装manpages-posix-dev就可以了
```
$sudo apt-get install manpages-posix manpages-posix-dev
```


