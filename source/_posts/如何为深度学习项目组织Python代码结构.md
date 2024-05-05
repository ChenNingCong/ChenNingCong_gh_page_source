---
title: 如何为深度学习项目组织Python代码结构
---

项目链接 https://github.com/ChenNingCong/deeplearning
在做深度学习的时候，我们往往会用到多种工具写Python代码，譬如vscode/jupyter nodebook，并且可能以多种多样的方式运行我们的代码，譬如直接在命令行运行，在jupyter notebook cell中运行等等，又或者我们想要打包这个代码以方便后续安装。

一个好的项目结构应该避免我们过多的设置环境变量或者不必要的一些操作（譬如改完代码以后必须按照包才能执行，譬如import各种报错）。在本文我总结了一下我个人喜欢的设置。

### &#x20;目录结构

存在这两种目录结构，一种是flat layout，另外一种是src layout。理论上说这两个没太大区别（因为后面我们都必须设置PYTHONPATH）。为了方便你可以用flat layout。以下是flat layout，如果要有src layout，只要把deeplearning/这个文件夹换成src/deeplearning就可以了（注意，src下必须还有一个deeplearning文件夹，然后所有代码放在这个deeplearning，不能直接src/）。**注意必须有一个\_\_init\_\_.py文件！**

如果你使用flat layout，那么需要在pyproject.toml中配置一个include参数，因为你可能有很多文件夹而包构建器不知道到底应该用哪个文件夹。

```Plain&#x20;Text
deeplearning
  -/deeplearning <-用来放我们的模型代码
    -/__init__.py <-必须有这个文件，不然安装以后没法import
    -/net
      -/net1.py
      -/net2.py 
    -/data
      -/dataloader.py
      -/dataset.py
    -... #其他代码
  -/script <- 一些训练用的脚本，可以是python或者script
  -/test <- 测试代码
pyproject.toml <-用来配置文件结构，便于打包安装
```

### 路径导入

在所有地方总是使用绝对导入，import deeplearning.XX， 不要用相对导入(from . import xxx)。

> 绝对路径+flat layout的好处很简单，首先包构建支持这种layout。其次vscode-python插件可以解析绝对导入，只要你的workspace root中使用flat layout或者src layout，vscode-python会自动把这个路径加入到import解析中中。如果你打开多个项目，就会加入多个import。 *最后一个好处是，当你在vscode用shift+enter逐行执行指令的时候，不会因为相对路径找不到顶层包而报错*。

### 脚本运行

为了运行脚本或者jupyter notebook，我们需要设置PYTHONPATH，这是因为我们使用绝对导入，而脚本在另外一个文件夹/script中，python解释器找不到，做法也很简单：

```shellscript
PYTHONPATH="." python script/xxx.py
```

PS:如果你跑集群的话，最好用绝对路径而不是相对路径。

如果你不想每次都设置PYTHONPATH，可以弄一个script/python\_launcher.sh脚本，然后在脚本里先设置PYTHONPATH，后运行程序。

对于vscode，我们可以设置.venv文件，这样GUI runner是可以自己检测PYTHONPATH，参照这个链接，https://stackoverflow.com/questions/67957883/how-can-i-set-in-vs-code-the-pythonpath-per-project

对于jupyter notebook，我还没想出什么好办法。上面那个加环境变量的做法是有用的，但是一般我不会只打开当前文件夹，而是打开一个父目录，这样子这个trick就用不了（因为环境变量必须在启动server前传进去）。我个人建议用vscode的notebook扩展而不是用jupyter notebook/lab，这样我们可以统一配置vscode不用搞两三个配置。而且在vscode里，pylance和AI扩展你都能直接用，不用额外配置。

这一大套问题本质上是Python可以有两种运行方式，一个是作为module，另外一个是作为script，vscode插件通过一些配置文件可以将这两种方式相结合，这样script也可以获得module的IDE支持，但像jupyter notebook这种交互运行的，就没这么好运了。



