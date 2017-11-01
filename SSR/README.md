# Ansible-SSR

    自学苦逼, 不写个项目出来都不敢说自己会配置管理! 

    这个项目的最终目的就是为了可以自动化安装一个多人翻墙SSR !

🔶 SSR 组成  

    前端: 一个网页. 可以用来注册, 查看流量
    后端: SSR 主程序搭在这里. 一台vps就是一个节点.用户通过节点服务器翻墙.
    数据库: 前端注册会把注册信息写到数据库中; 所以前端是要连数据库的.
            用户要连某个翻墙节点,节点(后端)首先要检查数据库有没有该用户.
            所以后端也是要连数据库的!

    三部分是可以分开的!!  也可以是一起的.
    单节点:  三个部分都安装在一起.
    多节点:  某个节点安装数据库/前端就可以了,  但是需要在所有节点安装后端.

    本文只介绍 SSR 后端的安装!! 而且你要用的话, 得自己改数据库信息! 
    我的数据库是 只有几个固定IP 可以连的. 你是连不上的.






# 目的 

数据库只有一个, 数据库要公开么! 
最好么... 只有一个数据库, 然后节点很多! 
后端是必须要连数据库才行的. 公开后怎么保证自己安全呢.能读. 能写



后端节点可以有很多.  
前端用




🔶 数据库安全

你可以连接 我的 数据库.
然后去我的 前端 注册....
然后 IP 用你自己的就可以了.
所以 你要的只是 搭建后端+然后连我的数据库.

你既然能连我的数据库...自然也能...




# TODO 

1. mysql 安装, 数据库创建. ssr.sql 脚本导入.

2. ssr 后端搭建 

3. ssr 前端搭建...






⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵 Ansible 实战 🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️

🔵 1. vps2 配置 ssh  让 gce 可以连上.


🔵 SSR 搭建简介

无非是一个从节点!  只要搭建下 ssr 后端就好了!  数据库/前端都不用!!

SSR 后端是需要加密的, 所以要安装依赖: libsodium

然后就是一些 python 依赖

最后就是 ssr 安装

然后 ssr 配置

再加上 服务的自启动! 就可以了!!!


🔵 libsodium 安装

1. 确保对方是 没安装 libsodium 的. 
然后我们用 playbook 来安装软件! 
gce 创建创建 ssr.yml


---
- hosts: vps2
  remote_user: root

  tasks:
  - name: ensure libsodium is installed
    yum: pkg=libsodium state=latest


✘✘∙GCE ~ ➜ ansible-playbook ssr.yml

TASK [ensure libsodium is installed] ***********************************************************************************
changed: [104.224.139.45]




然后去 vps2 上
yum list libsodium 就可以看到安装包了.说明安装好了.
我们再执行 yml 脚本就会发现, 这个任务返回的状态是 ok 而不是 changed,
说明vps2 已经安装过 这个依赖了! 自然也不需要改变了.
✘✘∙GCE ~ ➜ ansible-playbook ssr.yml

TASK [ensure libsodium is installed] ***********************************************************************************
ok: [104.224.139.45]



🔵 Python 依赖安装

    yum install python-setuptools -y 
    && yum install git 
    && easy_install pip 
    && pip install cymysql

主要是为了安装 cymysql 这个.

🔶  python-setuptools
  - name: python-setuptools
    yum : pkg=python-setuptools state=latest

    然后去 vps2 yum list python-setuptools 会发现有个 
    Installed Packages
    python-setuptools.noarch        0.9.8-7.el7        @base
    就说明安装好了.


🔶 Git 
    首先去 vps2 看看有没有安装 git ➜  yum list git 
    Installed Packages
    git.x86_64           1.8.3.1-6.el7_2.1           @base
    Available Packages
    git.x86_64           1.8.3.1-12.el7_4            updates
    发现以及安装了. 而且有个新版本出来了. 不管. 先卸载掉 yum remove git 
    再看 yum list git 就发现
    Available Packages
    git.x86_64           1.8.3.1-12.el7_4            updates
    也就说 当前没有安装! 


  - name: Git 
    yum : pkg=git state=latest

🔶 pip   

    easy_install pip

    首先得卸载啊. 

    在python的学习过程中，肯定会遇到很多安装模块的地方，
    可以使用easy_install安装，但是easy_install相对于pip而言，最大的缺陷就是它所安装的模块是不能够卸载的，其他功能是和pip一样的。
    卸载麻烦就不卸载了.那么怎么 用 easy_install 来安装 pip 呢!!!
    看官方文档就行了!  非常简单的...
    http://docs.ansible.com/ansible/latest/easy_install_module.html
    需要搜索什么 可以直接右下角进行搜索! 比如 pip 命令

    - easy_install:
        name: pip
        state: latest

    我们加个 name 上去 就变成了    !!! ?????? 

    - name:  install pip with easy_install
    easy_install: name=pip state=latest


🔶 cymysql

    这个是为了 python 可以连接 mysql 用的

    - name: install cymysql use pip
        pip: name=cymysql state=latest



🔶 安装 SSR 

    这个就是执行 git 命令了..
    这也是个模块!   而且也能指定安装路径!  原始命令是下面这样的
    🔅 cd ~ && git clone -b manyuser https://github.com/shadowsocksr/shadowsocksr.git

    丫的 原理 ssr 项目没有了啊 !!   fuck...  404 页面...
    丫的 ssr 项目终止了啊. 噩耗啊.. 哪里去找源码按....


    怎么转为 ansible 命令呢.
    首先要知道 clone 参数  和 -b 参数的作用

    ⦿ clone 参数
    下载就是 clone .上传就是 push ... 没啥好说的...

    参数挺多，但常用的就几个：

    1. 最简单直接的命令

    git clone xxx.git
    2. 如果想clone到指定目录

    git clone xxx.git "指定目录"

    - git:
        repo='https://github.com/Xu-Jian/shadowsocksr'
        dest=~/shadowsocksr

        这个项目失效了! 得换个链接才行!       
        好像不能直接安装到 /root 下的. 要给 git 项目一个文件夹!
        好像会进行初始化啊!  /root 目录当然不能初始化了.


🔶 SSR 初始化 

    下载后 是需要进入 ssr 文件夹 进行初始化的! 
    虽然只要这么一个命令!  cd ~/shadowsocksr && bash initcfg.sh
    其实这个命令也没干啥事... 就是设置权限. 加上把一些配置文件复制一份,
    方便你修改, 而是不是所有配置文件要你自己写....
    但是这个命令也不能重复运行! 不然你下面设置好的配置文件会被覆盖... 有重新执行..
    所以这个要怎么处理呢!!!
    只能事先判断了. 这个初始化命令会生成一个 usermysql.json.
    如果没有这个文件, 那么就进行初始化1  不然就不初始化! 

    ⦿  Ansible 文件判断 stat 模块
        Ansible 1有更直接的方式检查文件存在情况使用" stat " 模块。 

        当文件不存在. 就执行....初始化...

    tasks:
    - name: Check that the somefile.conf exists
        stat:
        path: ~/shadowsocksr/usermysql.json
        register: stat_result

    - name: Create the file, if it doesnt exist already
        shell: cd ~/shadowsocksr && bash initcfg.sh
        when: stat_result.stat.exists == False 





🔶 SSR 配置文件
    这个 只能事先 服务器上配置好! 然后自己给节点了!  
    为了统一, 我的配置文件就放到 github 上了.
    免得哪天 gce 重装了 啥也没有了. 

    userapiconfig.py
    连接前端的, 要配置 v2 还是v3 ,  单人ss 还是多人的 ssr 

    /root/shadowsocksr/usermysql.json
    连数据库用的! 

    /root/shadowsocksr/user-config.json
    配置后端的 加密方式! 

下一步就是下载配置文件了.
这里的话 文件肯定是存在的,,,, 要覆盖!也就是用服务器上的文件. 替换默认文件.





⦿ 数据库安全.
这个后端是要对数据库有读写权限的.
反正自己的 vps1 和  vps2 是固定IP 
所以 gce 的msql 可以设置成. 只允许从固定IP 访问.
并且权限的话 只能用某个数据库. 而不是所有的数据库.
aws 的 rds 数据库.... 

我们弄个永久免费的数据库.
RDS 的话只是... 12个月免费...也够了!  弄负载均衡.. 主从备份! 到期再说...

数据库安装好后 也是很多工作的....



🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶 数据库安装 🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶

怎么创建数据库呢...
RDS 不是 vps.. 好像没有登录功能的按! 怎么登录数据库呢...
对了 有个 域名. 然后账号密码就可以了.
但是怎么创建数据库了  mysql xxxx 命令 连到数据库就可以了.!
也能连上去... 

1. 首先建立数据库.  

2. 然后下载一个 db.sql 文件 (默认加密方式是 chacha20)
https://raw.githubusercontent.com/Xu-Jian/Ansible-DevOps/master/sources/db.sql

3. 导入数据表, 然后就算创建好了.
数据库不是重点.

把这个直接写成脚本不就可以了.
让脚本去 连接数据库. 下载配置文件. 执行文件! 
你可以用 shell  也能用python . 运维么. 还是 shell 的通用性强点. python 人家不一定有安装, 而且还要区分版本! 


要创建数据库

普通人是没有数据库权限的. 
所以数据库操作 最好是  写成 .sql ...
其实就是 sql 脚本...
里面都是一些数据库的命令! 为什么不自己写在 脚本里面呢! 
区分啊. 脚本太大了  也不知道干嘛的!  sql 脚本肯定是 操作 sql 的
怎么执行sql 文件呢... ssh 就可以. 
mysql –u用户名 –p密码 –D数据库<【sql脚本文件路径全名】
所以也不需要人工干预... 可以实现自动化的! 


下一步就是 ansible 判断数据库状态了... 能不能连上数据库
还好! 这个也已经有 模块了!!!  也不要你操心... 
mysql_db 模块, 配置下 主机,端口,账户密码.
这个能直接创建数据库. 当然执行 .sql 也不在话下!!!

⦿ 创建数据库

- name: Create a new database with name 'bobdata'
  mysql_db:
    name: bobdata
    state: present


⦿ 执行 sql 脚本...
- name: Import file.sql similar to mysql -u <username> -p <password> < hostname.sql
  mysql_db:
    state: import
    name: all
    target: /tmp/{{ inventory_hostname }}.sql








