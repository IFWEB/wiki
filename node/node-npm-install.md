## node安装 
环境是centOS
>wget https://nodejs.org/download/release/v6.10.0/node-v6.10.0-linux-x64.tar.xz  

或者手动下载然后拷贝到本地  
解压到当前目录：（推荐/usr/local）  
>xz -d node-v6.10.0-linux-x64.tar.xz  
>tar -xvf   node-v6.10.0-linux-x64.tar   

建立全局的软链接（{Node Path}是指node实际解压目录，如：/usr/local/node-v6.10.0-linux-x64）  
>ln -s /{Node Path}/bin/npm /usr/local/bin/   
>ln -s /{Node Path}/bin/node /usr/local/bin/  

验证    
>node -v

其他解压命令：    
1、*.tar 用 tar –xvf 解压   
2、*.gz 用 gzip -d或者gunzip 解压   
3、*.tar.gz和*.tgz 用 tar –xzf 解压   
4、*.bz2 用 bzip2 -d或者用bunzip2 解压   
5、*.tar.bz2用tar –xjf 解压   
6、*.Z 用 uncompress 解压   
7、*.tar.Z 用tar –xZf 解压   
8、*.rar 用 unrar e解压   
9、*.zip 用 unzip 解压   
  
   
  
## node、npm卸载
**方法一：单独卸载**  
卸载npm    
>sudo npm uninstall npm -g

可能会报“sudo: npm: command not found”，那么建议手动卸载“删除安装包”  

卸载node  
>yum remove nodejs npm -y 

无论这个方法成功与否，一般都需要用方法二查看有没有残留


**方法二：到目录中删除**  
进入 /usr/local/lib 删除所有 node 和 node_modules文件夹  
进入 /usr/local/include 删除所有 node 和  node_modules 文件夹  
进入 /usr/local/bin 删除 node 的可执行文件node和npm  

检查 ~ 文件夹里面的"local"   "lib"  "include"  文件夹，然后删除里面的所有  "node" 和  "node_modules" 文件夹  


## node 相关命令
查看设置NODE_ENV： echo $NODE_ENV; export NODE_ENV=test  


## npm 相关命令
查看npm环境变量：npm config list    
npm 安装路径: npm config get prefix; npm config set prefix *

