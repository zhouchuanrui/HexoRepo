# Hexo博客备份

这个repo就用作Hexo博客的同步repo用啦。

`.gitignore`用的就是默认的，在本地git测试之后，发现`hexo s`会出现乱码，也就是说在别的机上`git clone`之后还需要在博客目录中执行：

```
$ npm install hexo-renderer-ejs --save
$ npm install hexo-renderer-stylus --save
$ npm install hexo-renderer-marked --save          
```

安装这三个东西，然后才可以用。

