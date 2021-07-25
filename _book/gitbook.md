# git book相关命令  
```
gitbook  init
//运行
gitbook serve
//构建生成静态文件
gitbook build 
//当前目录生成指定文件pdf 
gitbook pdf . elasticsearch-book.pdf

nohup gitbook serve --port 5000  &
```
基础环境安装  
```
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
git clone git://github.com/creationix/nvm.git ~/nvm
command -v nvm
echo "source ~/nvm/nvm.sh" >> ~/.bashrc
source ~/.bashrc
nvm list-remote
nvm install v10.24.0
npm config set registry https://registry.npm.taobao.org
npm install -g cnpm --registry=https://registry.npm.taobao.org
sudo npm install gitbook-cli -g
npm install gitbook-cli -g
```

取消跟踪文件
```
 git rm -r --cached <目标文件夹>
```