node 版本更新：
1、首先需要清除原来node的 cache：
sudo npm cache clean -f

2、安装n模块
sudo npm install -g n 

安装成功我们就用使用n命令了

3、查看一下node所有的版本信息
 npm view node versions

我们可以选择任意一个版本:
sudo n 15.0.0

等安装完成 看一下版本即可：node -v
