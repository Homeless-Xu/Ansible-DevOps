# Ansible-SSR

    本文目的就是为了可以自动化安装一个多人翻墙SSR 的后端部分.

    2017-11-1-14 亲测可行  ✔︎


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



🔶 数据库安全

    我的 数据库密码都是直接写在配置文件中的.
    当然这个数据库的用户权限有限制的. 而且只能使用固定的IP 才能连. 
    下面就来说说 怎么操作..如果你要连我数据库,会有下面的报错. 
    错误码：1045  Access denied for user 'root@xx.xx.xx.xx'(using password:YES) 
    这个提示通常是由于Mysql默认的IP限制原因。 
    只要登录 mysql 把,某个用户的 host 的值 改为 % 就所有人都可以连了.
    把% 改成 127.0.0.1 就只有本机可连
    把% 改成 23.105.192.96 就只有这个IP可连



⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵 Ansible 实战 🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️

🔵 1. vps2 配置 ssh  让 gce 可以连上.


🔵 SSR 搭建步骤简介

    本文只搭建下 ssr 后端!  数据库/前端都不用!!
    SSR 后端是需要加密的, 所以要安装依赖: libsodium
    然后就是一些 python 依赖
    最后就是 ssr 安装
    然后 ssr 配置
    再加上 服务的自启动! 就可以了!!!


🔵 libsodium 安装

    🔶 gce 创建创建 ssr.yml

        ---
        - hosts: vps2
        remote_user: root

        tasks:
        - name: ensure libsodium is installed
            yum: pkg=libsodium state=latest

    🔶 GCE 运行脚本
        ✘✘∙GCE ~ ➜ ansible-playbook ssr.yml

        TASK [ensure libsodium is installed] ***************************************************
        changed: [104.224.139.45]

    🔶 vps2 检测

        yum list libsodium 可以看到安装包了.说明安装好了.
        我们再执行 yml 脚本就会发现, 这个任务返回的状态是 ok 而不是 changed,
        说明vps2 已经安装过 这个依赖了! 自然也不需要改变了.




🔵 依赖安装

🔶 简介
    
    安装 SSR 之前 需要安装下面这些依赖.

        yum install python-setuptools -y 
        && yum install git 
        && easy_install pip 
        && pip install cymysql


🔶  python-setuptools

    ⦿ playbook 脚本

        - name: python-setuptools
            yum : pkg=python-setuptools state=latest

    ⦿ vps2 验证:

     yum list python-setuptools 
     的 Installed Packages 里面如果有
     python-setuptools.noarch        0.9.8-7.el7        @base
     就说明安装好了.


🔶 Git 

    ⦿ vps2 查看

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

    ⦿ playbook 脚本

        - name: Git 
            yum : pkg=git state=latest


🔶 pip   

    pip 一般是用 easy_install pip 来安装的,
    这种安装方式 不好卸载. 我们就不卸载了.
    怎么用 Ansible 来安装 php呢! 用 easy_install 模块!
    模块不会用看官方文档就行了!  非常简单的...
    http://docs.ansible.com/ansible/latest/easy_install_module.html
    需要搜索什么其他模块, 可以直接右下角进行搜索! 比如 pip 命令

    官方手册的例子
    - easy_install:
        name: pip
        state: latest

    我们加个name 上去就可以了
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

    git 命令非常简单
    最简单直接的命令    git clone xxx.git
    如果想clone到指定目录    git clone xxx.git "指定目录"
    
    ansible 中的 git 模块, 默认值就是 clone . 所以可以不写.
    一般情况肯定是新建个文件夹来放 git 下载下来的文件的.
    所以要用 dest 参数来指定一个安装路径. 这个路径会自动创建.
    下面的 ~/ 目录下本来是没有shadowsocksr文件夹的, 
    执行这个 playbook 后会自动创建!

    - git:
        repo='https://github.com/Xu-Jian/shadowsocksr'
        dest=~/shadowsocksr




🔶 SSR 初始化 

    下载后 是需要进入 ssr 文件夹 进行初始化的! 
    虽然只要这么一个命令!  cd ~/shadowsocksr && bash initcfg.sh
    初始化脚本,也就是设置下文件权限. 加创建一些配置文件.
    但是这个命令也不能重复运行! 不然你下面设置好的配置文件会被覆盖... 
    这个初始化命令会生成一个 usermysql.json
    所以只要事先判断有没有这个文件, 就知道是否初始化过了.
    要判断文件状态 (检查文件存在) 就得用  stat 模块了

    - name: Check that the usermysql.json exists
        stat:
        path: ~/shadowsocksr/usermysql.json
        register: stat_result

    - name: do something, if the file doesn't exist
        shell: cd ~/shadowsocksr && bash initcfg.sh
        when: stat_result.stat.exists == False 


🔶 SSR 配置文件

    这个配置文件要么事先服务器上配置好! 然后复制给节点了!  
    要么就用正则式,改节点的配置文件.
    为了统一, 我的配置文件就放到 github 上了.免得哪天 gce 重装了 啥也没有了. 

    userapiconfig.py
    连接前端的, 要配置 v2 还是v3 ,  单人ss 还是多人的 ssr 

    /root/shadowsocksr/usermysql.json
    连数据库用的! 

    /root/shadowsocksr/user-config.json
    配置后端的 加密方式! 

    由于上面进行初始化后,这三个文件肯定是存在的,,,, 要先下载 然后 覆盖!
    下载模块了  get_url 
    http://docs.ansible.com/ansible/latest/get_url_module.html


    - name: Download usermysql.json
    get_url:
        url: https://raw.githubusercontent.com/Xu-Jian/Ansible-DevOps/master/SSR/sources/usermysql.json
        dest: /root/shadowsocksr/
        force: yes


    - name: Download userapiconfig.py
    get_url:
        url: https://raw.githubusercontent.com/Xu-Jian/Ansible-DevOps/master/SSR/sources/userapiconfig.py.json
        dest: /root/shadowsocksr/
        force: yes


    - name: Download user-config.json
    get_url:
        url: https://raw.githubusercontent.com/Xu-Jian/Ansible-DevOps/master/SSR/sources/user-config.json
        dest: /root/shadowsocksr/
        force: yes



🔶 启动服务

    - name: start ssr 
        shell: sh /root/shadowsocksr/run.sh


 🔶 开机自启  supervisorctl

    你可以用 rc.local 来自动启动. 也可以用 supervisor...
    andible 就有个 supervisor 模块!  supervisorctl ...

    - supervisorctl:
        name: my_app
        state: started

        当然这个得先安装并配置 supervisor ...


