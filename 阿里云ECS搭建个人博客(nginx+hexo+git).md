# 服务器个人博客部署与图床设置
> [知乎教程](https://zhuanlan.zhihu.com/p/120743882)

> [hexo主题](https://hexo.io/themes/)

> [git生成秘钥配置SSH公钥的简单方法](https://blog.csdn.net/xiayiye5/article/details/79652296?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.nonecase)

> [服务器安全防护](https://zhuanlan.zhihu.com/p/26282070)

## 服务器端软件配置
```
apt-get update
apt-get upgrade
apt-get install xfce4 # 简易桌面
apt-get install xfce4-terminal # vnc连接后不能打开终端问题
startx #启动xfce4
apt install vnc4server # vnc服务端
vnc4server:1 # 开启vncserver进程
vnc4server -kill :1 # 杀死服务进程，出现连接失败，可考虑重启vnc进程
apt-get install git nginx -y # 建议nginx源码安装
mkdir /var/repo/ # 建立存储库地址
chown -R $USER:$USER /var/repo/ # 修改权限
chmod -R 755 /var/repo/
cd /var/repo
git init --bare {自定义仓库名name}.git  # 建立远程存储库
mkdir -p /var/www/hexo # 创建nginx托管文件目录
chown -R $USER:$USER /var/www/hexo
chmod -R 755 /var/www/hexo

vim /etc/nginx/sites-available/default # 在server中修改，使root指向/var/www/hexo目录 'root /var/www/hexo;'；有域名可在server_name后写入域名
# 简单vim操作，i进入编辑模式，esc退出编辑模式，:wq保存退出。
service nginx restart # 重启nginx服务
# 此时测试搭建成功与否
在/var/www/hexo目录下，新建index.html，若不更改root指向，/var/www/html下有index.nginx-debian.html默认文件
<html>
<body>
<p>This is my Blog.</p>
</body>
</html>
# 访问公网网址
vim /var/repo/ganahBlog.git/hooks/post-receive # 创建钩子文件
#!/bin/bash
git --work-tree=/var/www/hexo --git-dir=/var/repo/ganahBlog.git checkout -f
chmod +x /var/repo/ganahBlog.git/hooks/post-receive

git clone root@服务器IP:/var/repo/ganahBlog.git or
git clone git@服务器IP:/var/repo/ganahBlog.git

配置_config.yml，将 url 改成https://服务器IP/，将 deploy 目标改为 服务器用户名@服务器IP:/var/repo/自定义存储库.git：
在个人博客站点目录下，打开 Git bash ,使用 hexo clean && hexo g -d 部署：
```
## 本地软件配置
```
sudo apt-get update
sudo apt-get install nodejs-dev node-gyp libssl1.0-dev
sudo apt-get install npm
mkdir hexo #创建目录，之后命令操作在该目录下
cd hexo #切换目录
sudo npm install -g hexo-cli
hexo init  ~/hexo #初始化 Hexo
gedit _config.yml
```
```
# _config.yml
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'

url: http://server-ip # 没有绑定域名时填写服务器的实际 IP 地址。
root: /
permalink: :year/:month/:day/:title/ 
or :category/:title/
permalink_defaults:
# Writing
new_post_name: :title.md # File name of new posts
default_layout: draft # 在source下产生_draft草稿，通过publish移入_post中
deploy:
    type: git
    repo: 你的服务器用户名@ECS云服务器的IP地址:/var/repo/hexo_static
    branch: master
```
```
hexo new 博客名称 #在hexo init的文件目录中
# 编辑md的题头及文章内容，按照格式操作
hexo publish 博客名称 #发布博客内容，移入_post中
```
## git部署本地博客到服务器
```
cd ~/hexo
npm install hexo-deployer-git --save #安装必要插件
hexo g -d #测试部署，若以root登录，则需要输入服务器密码，若git账户登录，输入自定义用户组密码，令牌待续..
```
## nginx开启图床设置
```
gedit /etc/nginx/nginx.conf
添加 'user root'
cd sites_available
gedit default 
在server 80 端口下新建,注意根目录不能明明images,否则与博客路径索引冲突
location /pictures/ {
								root /home/;
                                autoindex on;
}
service nginx stop
service niginx start
```
完成后既可以访问IP/pictures/的目录索引(不仅限于图片目录),图床引用IP/pictures/xxx.jpg
## 主题选择及主题优化
> [Hexo 搭建个人博客](http://yearito.cn/posts/hexo-theme-beautify.html)

> [Next主题](https://github.com/theme-next/hexo-theme-next)

> [Next动态插件](https://github.com/theme-next/theme-next-canvas-nest)

```
git clone https://github.com/theme-next/hexo-theme-next
# 放入theme中
git clone https://github.com/theme-next/theme-next-canvas-nest themes
# 不需要下载
# hexo/_config.yml
theme:next

# hexo/source/_data/footer.swig
<script color="0,0,255" opacity="0.5" zIndex="-1" count="99" src="https://cdn.jsdelivr.net/npm/canvas-nest.js@1/dist/canvas-nest.js"></script>

# theme/next/_config.yml
custom_file_path:
...
# 取消注释  footer: source/_data/footer.swig
# 添加以下代码
canvas_nest:
  enable: true
  onmobile: true # 是否在移动端显示
  color: '0,0,255' # 动态背景中线条的 RGB 颜色
  opacity: 0.5 # 动态背景中线条透明度
  zIndex: -1 # 动态背景的 z-index 属性值
  count: 99 # 动态背景中线条数量
```
## Some tricks
1. 查看服务器错误日志文件：/var/log/nginx/error.log
2. 建立远程连接ssh:
	```git
	adduser git # 按照指引输入密码等
    su git # 切换到git用户 git用户desktop下
    # 本地端生成ssh秘钥,切换到某一个账户下
    ssh-keygen -t rsa
    # 之后一路回车,产生秘钥的默认路径是:/home/账户/.ssh
    id_rsa(私钥) id_rsa.pub
    #  服务器端将密钥添加到 git 用户下,也可以直接添加root(不推荐)
    mkdir /home/git/.ssh # 或者~/.ssh 为root权限
	vim /home/git/.ssh/authorized_keys
    chmod 600 authorized_key (仅用户有读写权限)
    chomd 700 /home/账户/.ssh (仅用户自身有读写权限)
    vim /etc/ssh/sshd_config 修改一下两项
    PermitRootLogin yes 
    PasswordAuthentication yes 
	```
3. 初始化远程仓库使用git --bare init(instead of 'git init')
4. 文件互传scp(本质是ssh):
	- scp上传文件到服务器：scp 本地文件 用户名@IP:路径/文件
    - scp从服务器下载文件：scp 用户名@IP:路径/文件 本地文件
    - scp从服务器下载目录：scp -r 用户名@IP:路径/ 本地路径
    - scp上传文件到服务器：scp -r 本地路径 用户名@IP:路径/
5. 使用vim简单操作
三种模式切换:在命令模式下,i进入编辑模式,:进入末行模式,esc退回命令模式,wq保存退出,:q不保存退出