#### 前言
环境变量PATH（PATH环境变量是表示可执行文件的搜索路径），在命令行某个命令的时候,系统会到PATH对应的路径下查找可执行文件执行。
#### npm和node原理
安装node的时候有一个路径配置

![](https://github.com/IFWEB/wiki/blob/master/node/img/nodeandnvm/1.png)

表示node的安装路径是C:\Program Files\nodejs; 默认安装。node内置npm。
###### node的PATH:C:\Program Files\nodejs
###### npm的PATH:C:\Users\Administrator\AppData\Roaming\npm

![](https://github.com/IFWEB/wiki/blob/master/node/img/nodeandnvm/2.png)

可以看到npm的全局路径已经写到PATH，所以全局安装的可以直接执行对应的命令

在命令行运行npm实际是执行C:\Program Files\nodejs\npm 

我们在命令行执行：

    '/c/Program Files/nodejs/npm'

如下图所示
![](https://github.com/IFWEB/wiki/tree/master/node/img/nodeandnvm/3.png)

这个npm最终执行的文件是使用node运行'C:\Program Files\nodejs\node_modules\npm\bin\npm-cli.js'文件，可以做一个实验，我们将'C:\Program Files\nodejs\node_modules\npm‘文件改一个名称，

比如改成'npm1'，会发现执行npm报错 

![](https://github.com/IFWEB/wiki/tree/master/node/img/nodeandnvm/4.png)

如何查看npm的配置：

1. npm config list/ls //查看基本配置
2. npm config list/ls -l //查看所有配置


#### npm的两种安装模式

##### 本地安装
本地安装的特点：

1. 不会写入PATH变量（也就是环境变量，无法在全局引用该安装包，不能在终端直接使用）
2. 能够在不同的node_modules目录，安装不同版本的安装包 
3. 能够通过require()来引入安装包

##### 全局安装
全局安装的特点：

1. 不需要重复安装
2. 不能使用require()引入
3. 会写入PATH，并建立软链接，使用命令行的方式使用 
4. 不方便指定特定的版本运行

##### 两者的区别
1. npm install express // 本地安装，则是将模块下载到当前命令行所在目录。 
2. npm install -g express//全局安装，模块将被下载安装到【全局目录】中；

##### npm获取全局安装的路径
npm config get prefix

![](https://github.com/IFWEB/wiki/tree/master/node/img/nodeandnvm/5.png)

##### npm如何设置全局安装的默认目录？
npm config set prefix 'directory'

##### 查看包的安装路径
1. npm root （当前包的安装路径）
2. npm root -g （全局安装包路径）

![](https://github.com/IFWEB/wiki/tree/master/node/img/nodeandnvm/6.png)

##### 淘宝镜像
安装cnpm淘宝的npm镜像，可以加快npm包的安装。执行命令：
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
需要注意的是目前cnpm需要node 6.11.2以上版本，详细查看https://npm.taobao.org/

同理，之所以可以执行cnpm命令，实际是执行的C:\Users\Administrator\AppData\Roaming\npm\cnpm

npm全局安装以后之所以能识别全局安装后的包对应的命令是因为npm将全局安装路径写入了PATH，在执行npm安装的全局命令时系统会到PATH对应的路径下查找可执行文件执行。

#### nvm原理

**在开发中，有时候对node的版本有要求，有时候需要切换到指定的node版本来重现问题等。遇到这种需求的时候，我们需要能够灵活的切换node版本。
这里我们使用nvm工具来管理多版本node**

首先卸载node（如果之前有安装node）

**注：单独卸载node，npm的路径还在PATH环境变量中，只不过npm中安装的全局模块不能再访问了**

这里是图片

##### nvm的环境变量路径

    NVM_HOME :C:\Users\Administrator\AppData\Roaming\nvm

	NVM_SYMLINK：C:\Program Files\nodejs
	
	path:%NVM_HOME%;%NVM_SYMLINK%;

##### nvm安装路径

现在我安装的是v6.12.2/v8.8.0/v8.9.1


![](https://github.com/IFWEB/wiki/tree/master/node/img/nodeandnvm/7.png)

当我们启动某个node版本的时候，会在C盘的Program Files 创建一个映射文件nodejs
，并把文件映射到C:\Users\Administrator\AppData\Roaming\nvm

![](https://github.com/IFWEB/wiki/tree/master/node/img/nodeandnvm/8.png)

我们在对应的node版本中可以看到全局安装的npm包

![](https://github.com/IFWEB/wiki/tree/master/node/img/nodeandnvm/9.png)

我们使用nvm off 这个命令（关闭nodejs版本控制） 

C盘的Program Files 不存在nodejs的映射文件
