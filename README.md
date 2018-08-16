# 技术服务中心Wiki
该项目旨在整合技术服务中心的相关文档，提供一个友好的、易于学习和参考的资料库。
该Wiki主要涉及 **技术**，**项目** 和 **规范** 三块主题，在提交文档时也请根据内容选择恰当的主题进行提交。
- - - -

## 编译
本项目所有文档采用reStructText编写，可以通过 [sphinx](https://zh-sphinx-doc.readthedocs.io/en/latest/contents.html) 编译成html或pdf。因此需要保证在环境中安装了sphinx。在终端中输入：

```
  $ pip install sphinx
```

使用git clone下来该项目之后，在包含Makefile的那层目录中输入：

```
  $ make html
```

如果使用windows，在命令行中输入：

```
  $ make.bat html
```

会在当前目录中生成一个名字是build的文件夹，在浏览器中打开 docs/\_build/html/index.html 。


## 浏览

所有文档以在http://tech-service-wiki.readthedocs.io/可以访问！