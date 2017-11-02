# Ansible-DevOps

    自学苦逼, 不写个项目出来都不敢说自己会配置管理! 



# TODO 

# Done 

    • Ansible 搭建(服务端+节点)  2017-10-28-22 ✔︎

    • Ansible 基本介绍




## 参考

❗️❗️ Ansible中文权威指南 ❗️❗️
http://ansible-tran.readthedocs.io/en/latest/docs/intro_installation.html

★★★ ansible-first-book
https://www.gitbook.com/book/ansible-book/ansible-first-book


⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵 Ansible 搭建 🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️

🔵  Ansible 安装 CentOS_7 ✔︎

    yum install epel-release
    yum search ansible 
    yum install ansible -y
    ansible --version


🔵 Ansible 安装 Mac ✔︎

    🔶 Mac 安装

        其实服务端是不需要公有IP的.所以在Mac 本地搭服务会好很多...
        可以用brew 安装. 但是版本比较老.最好用 pip 安装. 
        sudo easy_install pip  ➜  安装 pip 
        sudo pip install ansible --quiet  ➜ 安装 ansible
        sudo pip install ansible --upgrade   ➜ 升级 ansible
        ansible --version ➜ 查看安装的版本

    🔶 Mac 配置

        配置文件不是必须的!!  但是主机文件是要有的!
        首先你要手动创建一个 /etc/ansible 文件夹
        然后你要 从别的地方下载一个 hosts 文件到上面的文件夹.
        然后配置 hosts. 之后就正常用了



🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶 密钥配置 ✔︎ 🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶

🔶 简介

    Ansible 之所以不要安装客户端 是因为服务端能直接SSH到所有的客户端!
    要实现这个功能,当然要事先配置 SSH 密钥了.
    也就是把 服务端的公钥 放到所有客户端上去.
    目的: 只要 Ansible 服务端 能不用密码直接ssh 到客户端就行. 不管任何方法.

🔶 密钥技巧

    其实一个人只要一套密钥就够了! 比如你用 Mac电脑生成一套密钥 (公钥+私钥)
    如果你把Mac 的公钥上传到 vps1 ➜ Mac  就可以SSH vps1
    如果你把Mac 的公钥上传到 vps2 ➜ Mac  就可以SSH vps2
    如果你把Mac 的私钥上传到 vps3 ➜ vps3 就可以SSH Mac + vps1 + vps2 了

    如果你有好几个VPS, 需要互相之间能直接SSH,
    那么最简单的方法是 Mac 本地生成一套密钥, 
    把这套密钥(公钥+私钥) 上传到 所有的VPS, 
    然后所有VPS 就可以互相SSH了


🔶 生成密钥 

    • 密钥生成 🔅 ssh-keygen -t rsa
    • 公钥位置 ~/.ssh/id_rsa.pub
    • 私钥位置 ~/.ssh/id_rsa  ⚡ 本文件非常重要,一般绝对不能外泄.

    ⚡ 不管是Mac 还是 Linux 生成密钥的命令都是一样的! 

🔶 密钥上传 +  authorized_keys  + 重启SSH 

    用 scp 命令把服务器的公钥传输到所有客户端上! 
    下面命令是把本地的密钥上传到服务器 root 用户的 ~/.ssh/ 目录下
    由于服务器是ssh 端口为了安全改成 2190 了 所以需要 -P 2190 参数

    当然 现在还不能互相SSH, 公钥上传只是第一步. 
    还要在服务器上把 公钥添加到 authorized_keys 里面.
    任何客户端想要ssh进服务器, 服务器首先查的就是这个 authorized_keys 文件. 
    如果这个文件里面 有客户端的公钥, 那么就允许SSH连接! 

    authorized_keys 这步是在对方服务器上设置的. 
    比如我Mac 要 SSH 到 GCE , 那么就去GCE设置这步!
    GCE是服务器. 然后很多员工都要SSH 到这个服务器. 所以GCE服务器上会有很多 公钥!
    因为每个员工都会生成一套自己的密钥对,私钥自己保存, 公钥上传到GCE 上面.
    每个员工都需要把自己的公钥 用下面命令追加到服务器GCE的 authorized_keys 文件中
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

    • GCE 服务器配置 (Ansible 安装在此)
    🔅 本地 ➜ scp -P 2190 -r ~/.ssh/id_rsa.pub root@35.194.128.92:~/.ssh/
    🔅 本地 ➜ scp -P 2190 -r ~/.ssh/id_rsa root@35.194.128.92:~/.ssh/
    🔅 远端 ➜ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    🔅 远端 ➜ /bin/systemctl restart  sshd.service

    • vps1 服务器配置
    🔅 本地 ➜ scp -P 2222 -r ~/.ssh/id_rsa.pub root@23.105.192.96:~/.ssh/
    🔅 本地 ➜ scp -P 2222 -r ~/.ssh/id_rsa root@23.105.192.96:~/.ssh/
    🔅 远端 ➜ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    🔅 远端 ➜ /bin/systemctl restart  sshd.service

    • vps2 服务器配置
    🔅 本地 ➜ scp -P 2222 -r ~/.ssh/id_rsa.pub root@104.224.139.45:~/.ssh/
    🔅 本地 ➜ scp -P 2222 -r ~/.ssh/id_rsa root@104.224.139.45:~/.ssh/
    🔅 远端 ➜ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    🔅 远端 ➜ /bin/systemctl restart  sshd.service

    • aws 服务器配置 ✔︎ 
    ⚡ AWS 默认是只能用 密钥 登录的, 而且这个密钥还是 AWS生成的.
    ⚡ 所以就不能在Mac 本地 用scp 命令把公钥上传到 aws
    ⚡ 而是要在 aws 用 scp 命令从某个有公网IP的电脑(比如 vps1) 下载密钥了!
    scp 上传命令语法 scp PathA user@IP:PathB   ➜ 把A 文件 上传到对方的B 路径下
    scp 下载命令语法 scp username@IP:pathB PathA ➜ 从对方B 文件夹 下载内容到本地的A

    🔅 AWS ➜ scp -P 2222 root@23.105.192.96:~/.ssh/id_rsa.pub ~/.ssh/
    🔅 AWS ➜ scp -P 2222 root@23.105.192.96:~/.ssh/id_rsa ~/.ssh/
    🔅 AWS ➜ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    🔅 AWS ➜ /bin/systemctl restart  sshd.service

    Mac 就能用 ssh root@13.115.230.116 登录 AWS了.
    当然最重要的是 Ansible 服务端 能ssh aws



🔶 SSH 测试

    ✘✘∙𝒗2 ~ ➜ ssh -p 2222 root@23.105.192.96
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
    Someone could be eavesdropping on you right now (man-in-the-middle attack)!
    It is also possible that a host key has just been changed.
    The fingerprint for the RSA key sent by the remote host is
    83:f4:2f:ae:7e:98:71:7c:95:42:ca:f6:88:27:9a:48.
    Please contact your system administrator.
    Add correct host key in /root/.ssh/known_hosts to get rid of this message.
    Offending RSA key in /root/.ssh/known_hosts:2
    RSA host key for [23.105.192.96]:2222 has changed and you have requested strict checking.
    Host key verification failed.

    这个错误就只要在本地 rm /root/.ssh/known_hosts 
    这个文件没啥用的! 第一次连服务器会记录对方的一些信息到本地而已.




🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶 Ansible 节点配置 ✔︎ 🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶

🔶 Why

    服务器必须要知道哪些节点是归自己管理的. 
    Ansible 服务端安装好后会有个 hosts 文件.
    修改这个配置来配置节点. vi /etc/ansible/hosts


🔶 如何配置 

    [webservers]
    alpha.example.org
    beta.example.org
    192.168.1.100
    192.168.1.110

    [vpses]                是组名.随意取 
    vps1 vps2              是主机名. 随意取
    ansible_ssh_host       指定服务器 ip
    ansible_ssh_port=2222  指定服务器ssh 端口
    ansible_ssh_user       指定服务器ssh 用户
    
    ⚡ 大概就是定义一个组来区分节点. 组下面可以写IP, 也能些域名.
    ⚡ 目的就是能连上节点服务器的ssh. 默认22端口的话 可以不写端口.
    ⚡ 如果SSH不是 22 端口, 就需要跟上端口号如:  23.105.192.96:2222


🔵 实际配置

    vi /etc/ansible/hosts

        [vps1]
        23.105.192.96:2222

        [vps2]
        104.224.139.45:2222

        [aws]
        13.115.230.116


🔵 节点测试 (在 ansible 服务端上进行)

    ⦿ ansible -m ping 'vps1' ✔︎
        23.105.192.96 | SUCCESS => {
            "changed": false,
            "failed": false,
            "ping": "pong"
        }

    ⦿ ansible -m ping 'vps2' ✔  ︎
        104.224.139.45 | SUCCESS => {
            "changed": false,
            "failed": false,
            "ping": "pong"
        }

    ⦿ ansible -m ping 'aws' 👹
        13.115.230.116 | FAILED! => {
            "changed": false,
            "failed": true,
            "module_stderr": "Shared connection to 13.115.230.116 closed.\r\n",
            "module_stdout": "/bin/sh: 1: /usr/bin/python: not found\r\n",
            "msg": "MODULE FAILURE",
            "rc": 0
        }

    为什么会报错呢! 明明 GCE上是可以直接SSH AWS的. 看返回的报错信息吧! 
    "module_stdout": "/bin/sh: 1: /usr/bin/python: not found\r\n",
    估计是python 没装! 因为 AWS是新系统.. 看看去..

🔶 Ubuntu 安装 Python 2.7 (忽略这步! Ubuntu 自带 Python 3.5 的)
    
    ★★★★★ 参考: https://vimsky.com/article/3577.html

    ⦿ 安装依赖!

        sudo apt-get install build-essential checkinstall

        sudo apt-get install libreadline-gplv2-dev libncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev

    ⦿ 下载 Python 

        version=2.7.13
        wget https://www.python.org/ftp/python/$version/Python-$version.tgz

    ⦿ 解压并cd到相应的目录

        tar -xvf Python-$version.tgz
        cd Python-$version

    ⦿ 用checkinstall 命令来安装,这中安装方式的好处是在需要时更容易卸载

        ./configure
        make
        sudo checkinstall

        **********************************************************************

        Done. The new package has been installed and saved to

        /root/Python-2.7.13/python_2.7.13-1_amd64.deb

        You can remove it from your system anytime using:

            dpkg -r python

        **********************************************************************

🔶 AWS 节点再次测试 👹

    还是一样的报错 
    那么估计是Python 路径问题了.
    CentOS 的 python 是 usr/bin/python
    ubuntu 的 Python 是 usr/local/bin/python     

    是不是创建个 快捷方式 就可以了呢...
    还是先谷歌下这个错误吧  "module_stdout": "/bin/sh: 1: /usr/bin/python: not found\r\n" 

    发现只要在 host 文件里面指定python 路径就可以了! 
    加两行, 大概就是 aws 这组, 添加一个变量: vars. 变量就是python2.7 的路径

    [aws:vars]
    ansible_python_interpreter=/usr/local/bin/python

🔶 AWS 节点再次 ping 测试 ✔︎

    ✘✘∙GCE ansible ➜ ansible -m ping 'aws'
    13.115.230.116 | SUCCESS => {
        "changed": false,
        "failed": false,
        "ping": "pong"
    }    


🔶 AWS 节点 内存使用情况

    ✘✘∙GCE ansible ➜ ansible -m shell -a 'free -m' 'aws'
    13.115.230.116 | SUCCESS | rc=0 >>
                total        used        free      shared  buff/cache   available
    Mem:            990          75         257           4         658         711
    Swap:             0           0           0

    ⚡ 到这里Ansible 就搭建好了! 没错! 就搭建好了.... 剩下的就是怎么使用了



⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵 Ansible 基础知识 🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️

🔶 简介

    只要 Ansible 能成功连上客户端, 那么就算搭建成功了! 非常简单的!
    当然会搭建 不等于说你会 Ansible 了...  连皮毛都还没学会呢..继续...
    通过上面的搭建,对Ansible 还是没啥了解! 首先我们先学下 Ansible 基础! 


🔶 配置管理

    配置管理系统旨在使管理员和操作团队能够轻松地控制大量服务器。
    它们允许您从一个中心位置以自动方式控制许多不同的系统。

    配置管理那么多: Puppet、Ansible、Chef....
    Puppet 太重量级了,而且蛮复杂的! 
    谷歌很多年前就用 Puppet 管理6000+台 Mac 桌面电脑.
    适合人家大公司的不一定适合中小公司!!!
    Ansible 应该是中小公司最佳的选择. Chef.. 没啥了解...
    自己这水平也进不了大公司,于是就选择了 Ansible.


🔶 Ansible 
    
    Ansible 自动化运维工具!  基于Python开发的.
    非常轻量! 搭建只需下面两步!  无须在被控主机安装任何软件. 
        1. 管理节点安装 Ansible, 
        2. 管理节点配置被控主机的 SSH 就算搭好了.

    Ansible 不需要在节点中安装任何客户端。它使用SSH来和节点进行通信.
    Ansible 支持多种系统! Redhat、Debian、Windows 都可以!

    理论上. 只要能通过SSH 干的事情, Ansible 都能实现.
    拷贝文件、安装软件包、启动服务...

    Ansible 是基于模块工作的!!!  
    Ansible 本身没有啥功能,就是个框架!提供些基础服务. 重点是模块!!!!

    你可以使用 Ansible 内置命令 来管理主机.
    你可以使用 Ansible 脚本语言 来管理主机.  ➜ Playbook
    
    你想让 Absible 去 ping 某个主机,
    你可以自己用 Ansible 内置的命令,你也可以自己写 palybook 来实现!


🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶 Ansible 命令   🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶

🔶 简介

    Ansible 自带一些基础命令! 
    比如 执行命令、拷贝文件、安装软件包、添加用户、启动服务、查看详细系统信息....
    其实现在没必要仔细学这个, 暂时的话能看懂就好.

🔶 基础用法 1

    ansible -m ping all         所有主机
    ansible -m ping vpses       指定一个组
    ansible -m ping vps1        指定单个主机  
    ansible -m ping vps1:vps2   指定多个主机

    例如，要找出我们的host1机器上的内存使用情况，我们可以使用：
    ansible -m shell -a 'free -m' host1


🔶 基础用法 2

    ⦿ 检查ansible安装环境

        检查所有的远程主机，是否以bruce用户创建了ansible主机可以访问的环境。
        $ansible all -m ping -u bruce

    ⦿ 执行命令

        在所有的远程主机上，以当前bash的同名用户，在远程主机执行“echo bash”
        $ansible all -a "/bin/echo hello"

    ⦿ 拷贝文件

        拷贝文件/etc/host到远程主机（组）web，位置为/tmp/hosts
        $ ansible web -m copy -a "src=/etc/hosts dest=/tmp/hosts"

    ⦿ 安装包

        远程主机（组）web安装yum包acme
        $ ansible web -m yum -a "name=acme state=present"

    ⦿ 添加用户

        $ ansible all -m user -a "name=foo password=<crypted password here>"

    ⦿ 下载git包

        $ ansible web -m git -a "repo=git://..."

    ⦿ 启动服务

        $ ansible web -m service -a "name=httpd state=started"

    ⦿ 并行执行

        启动10个并行进行执行重起
        $ansible lb -a "/sbin/reboot" -f 10

    ⦿ 查看远程主机的全部系统信息！！！

        $ ansible all -m setup


🔶 基础用法 3

    ⦿ 检查Ansible节点的运行时间（uptime):
    ansible -m command -a "uptime" 'dbservers'

    ⦿ 检查节点的内核版本
    ansible -m command -a "uname -r" 'dbservers'

    ⦿ 给节点增加用户
    ansible -m command -a "useradd david" 'dbservers'

    ⦿ 重定向输出到文件中
    ansible -m command -a "df -Th" 'dbservers' > /tmp/command-output.txt
    cat /tmp/command-output.txt




🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶 Playbook 简介 🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶

🔶 Playbook

    说到自动化,肯定是离不开脚本的! 
    Ansible 也提供了 脚本功能. Ansible 的脚本有个名字 叫 playbook.
    playbook 使用的是 YAML 格式, Playbook 脚本文件以 yml 结尾. YAML 类似 JSON. 


🔶 执行 Playbook 的方法

    ansible-playbook deploy.yml


🔵 Playbook 实例

    ---
    - hosts: web
    vars:
        http_port: 80
        max_clients: 200
    remote_user: root
    tasks:
    - name: ensure apache is at the latest version
        yum: pkg=httpd state=latest

    - name: Write the configuration file
        template: src=templates/httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
        notify:
        - restart apache

    - name: Write the default index.html file
        template: src=templates/index.html.j2 dest=/var/www/html/index.html

    - name: ensure apache is running
        service: name=httpd state=started
    handlers:
        - name: restart apache
        service: name=httpd state=restarted



    这个playbook的功能是为web主机部署apache, 包含以下部署步骤：
    • 安装apache包；
    • 拷贝配置文件httpd，并保证拷贝文件后，apache服务会被重启；
    • 拷贝默认的网页文件index.html；
    • 启动apache服务；


    playbook deploy.yml包含下面几个关键字，每个关键字的含义：+

    hosts：为主机的IP，或者主机组名，或者关键字all
    remote_user: 以哪个用户身份执行。
    vars： 变量

    tasks: playbook的核心，定义顺序执行的动作action。
    每个action调用一个ansbile module。
    action 语法： module： module_parameter=module_value
    常用的module有yum、copy、template等...

    handers： 是playbook的event，默认不会执行，在action里触发才会执行。
    多次触发只执行一次。



🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶 Ansible 模块 🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶

🔶 简介

    模块非常强大的, 很多功能不需要你自己写脚本,可以直接实现某功能.
    比如软件的安装, 比如 配置文件的分发, 比如服务的控制!!!
    你可以在Ansible 命令里用模块, 也能在 Playbook 里用模块.


🔶 模块 (Ansible 命令)

    ⦿ 语法: 
        -m + 模块名
        -a + 模块参数

    ⦿ 例: 
        用 copy 模块把管理员节点的 /etc/hosts文件 复制到所有远程主机/tmp/hosts
        $ ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"

        用 yum 模块在名为 web 的远程主机上安装httpd包
        $ ansible web -m yum -a "name=httpd state=present"


🔶 模块 (Ansible Playbook)

    ⦿ 语法
        在playbook脚本中
        tasks中的每一个action都是对module的一次调用。
        在每个action中：冒号前面是module的名字; 冒号后面是调用module的参数

        ---
        tasks:
        - name: ensure apache is at the latest version
            yum: pkg=httpd state=latest
        - name: write the apache config file
            template: src=templates/httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
        - name: ensure apache is running
            service: name=httpd state=started
            

🔶 常用模块

    有些模块是 Ansible 非常常用的! 如: 
        • yum 模块可以在节点安装软件!
        • template 模块可以把服务器上的某些文件 发送到客户端! 
        • service 模块可以控制服务.


    🔶调试和测试类的module

        ping - ping一下你的远程主机，如果可以通过ansible成功连接，那么返回pong
        debug - 用于调试的module，只是简单打印一些消息，有点像linux的echo命令。


    🔶 文件类的module

        copy - 从本地拷贝文件到远程节点
        template - 从本地拷贝文件到远程节点，并进行变量的替换
        file - 设置文件的属性


    🔶 linux上常用的操作

        user - 管理用户账户
        yum - red hat系linux上的包管理
        service - 管理服务
        firewalld - 管理防火墙中的服务和端口


    🔶 执行Shell命令

        shell - 在节点上执行shell命令，支持$HOME和"<", ">", "|", ";" and "&"
        command - 在远程节点上面执行命令，不支持$HOME和"<", ">", "|", ";" and "&"





🔵 Ping 模块 ➜ 节点测试

    最常用的测试模块, 测试一个节点有没有 配置好SSH.
    ping 模块 可不是 Linux 下的 ping 命令! 强大很多.
    Ping 模块首先检查能不能SSH登录!  然后还会检查对方Python版本是否满足Ansible的要求.
    如果满足要求 就会返回ping, 不满足就会报错! 

    ✘✘∙GCE ~ ➜ ansible aws -m ping
    13.115.230.116 | SUCCESS => {
        "changed": false,
        "failed": false,
        "ping": "pong"
    }


🔵 Copy 模块 ➜ 文件复制

    这个模块可以把服务器上的静态文件, copy 到远程节点上! 而且还能给文件设置权限! 
    大概原理就是 首先对比双方文件的 checksum 
    如果一致 说明远程的文件是最新的, 无需copy
    如不一致 要么远程没有这个文件,要么文件版本太旧. 会直接复制/替换.
    当然保险一点, 你也可以选择 先备份对方的文件,再进行copy.


    ⦿ 设置文件权限 

        把本地 myfiles 下的配置文件 复制到远程的 etc 目录下.
        并设置 所属用户、用户组、用户权限

        - copy:
            src: /srv/myfiles/foo.conf
            dest: /etc/foo.conf
            owner: foo
            group: foo
            mode: 0644


    ⦿ 先备份再复制

        backup参数为yes的时候，如果发生了拷贝操作，那么会先备份下目标节点上的原文件。
        当两个文件相同时，不会进行拷贝操作，当然也没有必要备份啦。

        - copy:
            src: sudoers
            dest: /tmp
            backup: yes


    ⦿ Copy + 验证操作
        如果你觉得仅仅复制文件的功能还不够,比如某些配置文件, 你需要检查一下语法!!
        哪怕本地检查过了,远程再检查一遍也是好的.那么你可以加个验证参数,
        这样的话, 不仅仅需要文件拷贝成功, 还需要验证命令返回成功值才行.
        visudo -cf /etc/sudoers是验证sudoers文件有没有语法错误的命令。

        - copy:
            src: /mine/sudoers
            dest: /etc/sudoers
            validate: 'visudo -cf %s'
            

🔵 Template 模块 ➜ 文件模板

    这个也是复制文件用的. 和copy 差不多, 但是比copy 强大很多.
    copy 只适合那些 静态文件!  不需要改变的文件.
    但是事实上.很多文件都需要对服务器的模板文件进行一些修改后再复制到节点.
    这时候你就要用到 Template 模块了.

    比如我要给所有的远程节点都发送一个 Apache 的测试文件! 一个 index.html
    这个index.html 文件的功能是显示当前节点的主机名和IP地址.
    那么这个 index.html 肯定不是静态文件了... 
    这里就需要用到模板了! 如果你接触过模板 那么很好理解.
    大概就是设置几个变量. 然后按照模板的语法就可以了.很简单的.

    这里也一样! 主机名 和 IP 都是变量! 
    我们的 index.html 只要按照下面这么写就可以了
    就用了两个变量而已. ansible_hostname 和  ansible_default_ipv4.address
    变量外面的 {{}} 是模板语法, 就是这个文件再发送给对方之前! 
    Template 模块 会首先把  {{ ansible_hostname }} ({{ ansible_default_ipv4.address }})
    变成对方的真实 主机名 和 真实IP.

    <html>
    <title>Demo</title>
    <body>
    <div class="block" style="height: 99%;">
        <div class="centered">
            <h1>#46 Demo</h1>
            <p>Served by {{ ansible_hostname }} ({{ ansible_default_ipv4.address }}).</p>
        </div>
    </div>
    </body>
    </html>

    ➜  真正发给对方的是类似下面的

    <html>
    <title>Demo</title>
    <body>
    <div class="block" style="height: 99%;">
        <div class="centered">
            <h1>#46 Demo</h1>
            <p>Served by GCE (35.194.128.92).</p>
        </div>
    </div>
    </body>
    </html>

    再来说说那两个变量, 这个是 Ansible 自带的变量. 所以变量名不能乱改的.
    当然你也可以 使用普通变量 (自己设置变量). 
    普通变量的值 就需要你自己设置了!!!  在 playbook 中用 vars 来定义变量.

    - hosts: localhost
    vars:
        http_port: 8080
    remote_user: root
    tasks:

    - name: Write the configuration file
        template: src=templates/httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf

    然后你就可以在 httd.conf.j2 中使用 http_port 这个变量了, 比如.
    Listen {{ http_port }}



🔵 File 模块 ➜ 文件操作

    设置远程主机 文件/文件夹 权限、 创建/删除文件....

    🔶 创建文件 

    - file:
        path: /etc/foo.conf
        state: touch
        mode: "u=rw,g=r,o=r"

    🔶 创建文件夹

        - file:
            path: /etc/some_directory
            state: directory
            mode: 0755    


🔵 User 模块
    
    user module可以增、删、改Linux远程节点的用户账户，并为其设置账户的属性。+

    🔶 增加账户

        增加账户johnd，并且设置uid为1040，设置用户的primary group为admin
        - user:
            name: johnd
            comment: "John Doe"
            uid: 1040
            group: admin
        创建账户james，并为james用户额外添加两个group
        - user:
            name: james
            shell: /bin/bash
            groups: admins,developers
            append: yes


    🔶 删除账户

        删除账户johnd
        - user:
            name: johnd
            state: absent
            remove: yes

    🔶 修改账户的属性

        为账户jsmith创建一个 2048-bit的SSH key，放在~jsmith/.ssh/id_rsa
        - user:
            name: jsmith
            generate_ssh_key: yes
            ssh_key_bits: 2048
            ssh_key_file: .ssh/id_rsa
        为用户添加过期时间：
        - user:
            name: james18
            shell: /bin/zsh
            groups: developers
            expires: 1422403387



🔵 yum ➜ 软件安装

    yum module是用来管理red hat系的Linux上的安装包的，包括RHEL，CentOS...

    🔶 从yum源上安装和删除包

        安装最新版本的包，如果已经安装了老版本，那么会更新到最新的版本：
        - name: install the latest version of Apache
        yum:
            name: httpd
            state: latest
        安装指定版本的包
        - name: install one specific version of Apache
        yum:
            name: httpd-2.2.29-1.4.amzn1
            state: present
        删除httpd包
        - name: remove the Apache package
        yum:
            name: httpd
            state: absent
        从指定的repo testing中安装包
        - name: install the latest version of Apache from the testing repo
        yum:
            name: httpd
            enablerepo: testing
            state: present


    🔶 从yum源上安装一组包

        - name: install the 'Development tools' package group
        yum:
            name: "@Development tools"
            state: present

        - name: install the 'Gnome desktop' environment group
        yum:
            name: "@^gnome-desktop-environment"
            state: present


    🔶 从本地文件中安装包

        - name: install nginx rpm from a local file
        yum:
            name: /usr/local/src/nginx-release-centos-6-0.el6.ngx.noarch.rpm
            state: present


    🔶 从URL中安装包

        - name: install the nginx rpm from a remote repo
        yum:
            name: http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
            state: present



🔵 service 模块 ➜ 服务控制

    管理远程节点上的服务，开、关、重起、重载服务
    什么是服务呢，比如httpd、sshd、nfs、crond等。

    🔶 开httpd服务

        - service:
            name: httpd
            state: started

    🔶 关服务

        - service:
            name: httpd
            state: stopped

    🔶 重起服务

        - service:
            name: httpd
            state: restarted

    🔶 重载服务

        - service:
            name: httpd
            state: reloaded

    🔶 设置开机启动的服务

        - service:
            name: httpd
            enabled: yes

    🔶 启动网络服务下的接口

        - service:
            name: network
            state: restarted
            args: eth0




🔵 firewalld 模块

    firewalld module为某服务和端口添加firewalld规则。
    firewalld中有正在运行的规则，和永久的规则，firewalld module都支持。
    firewalld要求远程节点上的firewalld版本在0.2.11以上。

    🔶 为服务添加firewalld规则

        - firewalld:
            service: https
            permanent: true
            state: enabled
        - firewalld:
            zone: dmz
            service: http
            permanent: true
            state: enabled

    🔶 为端口号添加firewalld规则

        - firewalld:
            port: 8081/tcp
            permanent: true
            state: disabled
        - firewalld:
            port: 161-162/udp
            permanent: true
            state: enabled

    🔶 其它复杂的firewalld规则

        - firewalld:
            rich_rule: 'rule service name="ftp" audit limit value="1/m" accept'
            permanent: true
            state: enabled
        - firewalld:
            source: 192.0.2.0/24
            zone: internal
            state: enabled
        - firewalld:
            zone: trusted
            interface: eth2
            permanent: true
            state: enabled
        - firewalld:
            masquerade: yes
            state: enabled
            permanent: true
            zone: dmz


🔵 Shell 模块 ➜ 执行命令

    通过/bin/sh在远程节点上执行命令。

    如果一个操作你可以通过Module yum，copy操作实现，那么你不要使用shell或者command这样通用的命令module。

    因为通用的命令module不会根据具体操作的特点进行status的判断，
    所以当没有必要再重新执行的时候，它还是要重新执行一遍。
    支持$home，支持$HOME和"<", ">", "|", ";" and "&"。

    🔶 支持$home ➜  shell: echo "Test1" > ~/tmp/test1
    🔶 支持&&    ➜  - shell: service jboss start && chkconfig jboss on
    🔶 支持>>    ➜  - shell: echo foo >> /tmp/testfoo

    🔶 调用脚本
        - shell: somescript.sh >> somelog.txt

    🔶 执行命令前，改变工作目录
        - shell: somescript.sh >> somelog.txt
        args:
            chdir: somedir/

    🔶 在执行命令前改变工作目录，且仅在文件somelog.txt不存在时才执行该action。
        - shell: somescript.sh >> somelog.txt
        args:
            chdir: somedir/
            creates: somelog.txt

    🔶 指定用bash运行命令
        - shell: cat < /tmp/\*txt
        args:
            executable: /bin/bash







🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶 Ansible 配置 🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶
        任何软件都要配置..  Ansible 当然也是...你需要有个大概的了解! 

⦿ 主机清单文件:            ➜ inventory  = /etc/ansible/hosts 
⦿ 额外模块放置路径         ➜ library    = /usr/share/my_modules/
⦿ 远程主机的临时文件位置   ➜ remote_tmp = $HOME/.ansible/tmp
⦿ 管理节点上临时文件的位置 ➜ local_tmp  = $HOME/.ansible/tmp



🔵 Host Inventory（主机清单）

    主机清单，告诉ansible需要管理哪些server，和server的分类和分组信息。
    可以根据你自己的需要根据地域分类，也可以按照功能的不同分类。

    主机清单默认文件位置  /etc/ansible/hosts


    🔶 简单的分组[]内是组名

        mail.example.com

        [webservers]
        foo.example.com
        bar.example.com

        [dbservers]
        one.example.com
        two.example.com
        three.example.com

        [webservers]
        www[01:50].example.com

        [databases]
        db-[a:f].example.com


🔶 指定连接参数

    [targets]

    localhost              ansible_connection=local
    other1.example.com     ansible_connection=ssh        ansible_user=mpdehaan
    other2.example.com     ansible_connection=ssh        ansible_user=mdehaan


 🔶 变量  
    
    为一个组指定变量

    [atlanta]
    host1
    host2

    [atlanta:vars]
    ntp_server=ntp.atlanta.example.com
    proxy=proxy.atlanta.example.com



🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶 Playbook 详解 🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶

🔶 别人的Playbook

    能够学会快速用别人的成果，高效地分享自己的成果，才是好码农.
    在你动手从头开始写一个Playbook之前，不如先参考一下别人的成果吧。

    ⦿ 官方例子

        Ansible官方提供了一些比较常用的、经过测试的Playbook例子：
        https://github.com/ansible/ansible-examples

    ⦿ Playbook分享平台

        此外，Ansible还提供了Playbook的分享平台，上面的例子是Ansible用户自己上传的。
        你如果在没有思路的情况下参考下吧，不过一定要再重新严谨的测试下。
        https://galaxy.ansible.com/


🔶 Playbook 基本语法

    运行脚本 ➜    ansible-playbook deploy.yml
    脚本详细输出➜   ansible-playbook playbook.yml  --verbose
    查看脚本影响哪些hosts➜   ansible-playbook playbook.yml --list-hosts
    并行执行脚本  ➜  ansible-playbook playbook.yml -f 10


🔶 Playbook 组成 

    最基本的 playbook 脚本分为三部分.

    ⦿ 在什么机器上,用什么身份执行 ➜   hosts  users 
    ⦿ 执行哪些任务  ➜  tasks
    ⦿ 善后的任务 ➜  handlers 


🔶 deploy.yml 例子

    ---
    - hosts: webservers
    vars:
        http_port: 80
        max_clients: 200
    user: root
    tasks:
    - name: ensure apache is at the latest version
        yum: pkg=httpd state=latest
    - name: write the apache config file
        template: src=/srv/httpd.j2 dest=/etc/httpd.conf
        notify:
        - restart apache
    - name: ensure apache is running
        service: name=httpd state=started
    handlers:
        - name: restart apache
        service: name=httpd state=restarted


🔵 主机和用户 

hosts	为主机的IP，或者主机组名，或者关键字all
user	在远程以哪个用户身份执行。


🔵 Tasks任务列表

    tasks是从上到下顺序执行，如果中间发生错误，那么整个playbook会中止。

    每一个task是对module的一次调用。使用不同的参数和变量而已。

    每一个task最好有name属性，这个是供人读的，没有实际的操作。
    然后会在命令行里面输出，提示用户执行情况。


🔶 task的基本写法:

    tasks:
    - name: make sure apache is running
        service: name=httpd state=running

    其中name是可选的，也可以简写成下面的例子。

    tasks:
    - service: name=httpd state=running

    写name的task在playbook执行时，会显示对应的名字，信息更友好、丰富。
    写name是个好习惯！
    TASK: [make sure apache is running] *************************************************************
    changed: [yourhost]

    没有写name的task在playbook执行时，直接显示对应的task语法。
    在调用同样的module多次后，不好分辨执行到哪步了。
    TASK: [service name=httpd state=running] **************************************
    changed: [yourhost]


🔶 参数写法  key=value

    tasks:
    - name: make sure apache is running
        service: name=httpd state=running

    当需要传入参数列表太长时，可以分隔到多行：
    
    tasks:
    - name: Copy ansible inventory file to client
        copy: src=/etc/ansible/hosts dest=/etc/ansible/hosts
                owner=root group=root mode=0644

    或者用yml的格式传入参数
    tasks:
    - name: Copy ansible inventory file to client
        copy:
        src: /etc/ansible/hosts
        dest: /etc/ansible/hosts
        owner: root
        group: root
        mode: 0644
        


🔶 TASK的执行状态

    task中每个action会调用一个module，
    在module中会去检查当前系统状态是否需要重新执行。

    如果本次执行了，那么action会得到返回值changed;
    如果不需要执行，那么action得到返回值ok

    module的执行状态的具体判断规则由各个module自己决定和实现的。
    例如，"copy" module的判断方法是比较文件的checksum


🔵 响应事件 Handler 

    每个主流的编程语言都会有event机制，那么handler就是playbook的event。

    Handlers里面的每一个handler，也是对module的一次调用。 但是 handlers与tasks不同，
    tasks会默认的按定义顺序执行每一个task，
    handlers则不会，它需要在tasks中被调用，才有可能被执行。

    Tasks中的任务都是有状态的，changed或者ok。 
    在Ansible中，只在task的执行状态为changed的时候，才会执行该task调用的handler，
    这也是handler与普通的event机制不同的地方。


🔶 Handler 应用场景

    什么情况下使用handlers呢?

    如果你在tasks中修改了apache的配置文件。需要重起apache。
    此外还安装了apache的插件。那么还需要重起apache。 像这样的应该场景中，
    不要在每个 task 后面 立即重启 Apache. 而是把 重起apache 设计成一个handler.
    这个 Handler 放在所有 Tasks 后面. 
    用了Handle 只要重启一次就好了... 而不是 重启很多次...


🔶 一个handler最多只执行一次
    在所有的任务里表执行之后执行
    如果有多个task notify同一个handler,那么只执行一次。





🔵 变量

用户可以在Playbook中，通过vars关键字自定义变量，使用时用{{ }}引用以来即可。+


🔶 主机的系统变量 facts 

    ansible 会通过 setup 模块来收集主机的系统信息! 
    这些收集到的信息叫做 facts.
    这些 facts 是可以直接用变量的形式使用的.

    你用 ansible aws -m setup 命令就会输出 aws 收集到的信息! 非常的多!

    "ansible_hostname": "ip-172-31-18-193",

    "ansible_eth0": {
                "active": true,
                "device": "eth0",
                "hw_timestamp_filters": [],
                "ipv4": {
                    "address": "172.31.18.193",
                    "broadcast": "172.31.31.255",
                    "netmask": "255.255.240.0",
                    "network": "172.31.16.0"
                },


    这些变量都是能直接用的!!!  某些变量还是非常有用的.
    上面的变量分两种! 一种简单的. 只有一行, 
    一种复杂的, 比如 IPv4 这个变量 里面有好几个值.

    简单变量直接用 比如:  {{ansible_hostname}}

    如果我想获取 ip 地址怎么办呢, 加个[] 就行!  或者用.号(推荐)
    {{ ansible_eth0.ipv4.address }}
    {{ ansible_eth0["ipv4"]["address"] }}



🔶 命令行传递变量.

    playbook 允许用户在执行的时候传入变量值.
    当然 变量的定义还是要在 playbook 里定义的. 
    比例 

    在release.yml文件里，hosts和user都定义为变量，
    脚本里面先不指定变量值! 我们从命令行传递变量值。
    ---

    - hosts: '{{ hosts }}'
    remote_user: '{{ user }}'

    tasks:
        - ...

    然后你就可以这样传入值了  也就是用 --extra-vars 额外变量!

    ansible-playbook e33_var_in_command.yml --extra-vars "hosts=web user=root"


🔶



🔶




🔶



🔵 模块分类

    Core Module  和 Extra module 

    核心模块 不需要下载. 经过严格测试,直接能用的! 
    额外模块 需要下载并配置, 有可能有BUG.



🔵 Playbook 脚本指南 

    1. 尽量用include 和 role  避免重复的代码
    2. 尽量把大的文件分成小的文件.

一个 playbook 由一个或多个 paly 组成! 
每个play 定义了一系列的 task,
其中每个 task 一般就是调用一个 ansible 模块.

例子：只带有一个play的playbook文件test.yml：


---               #任何playbook文件(其实就是yaml文件)都要以这个开头
- hosts: webservers         #对webservers主机组下的所有主机进行操作
  vars:           #为该play定义两个变量
    http_port: 80
    max_clients: 200
  remote_user: sapser       #连接到远程主机的用户
  sudo: yes       #以sudo模式运行该play
  sudo_user: root           #sudo到哪个用户，默认为root，如果sudo到该用户需要密码，则在执行ansible-playbook的时候指定-K选项来输入sudo密码
  tasks:          #开始定义task
  - name: ensure apache is at the latest version            #这既是每个task的说明也是每个task的名字
    yum: pkg=httpd state=latest    
    tags:         #给该task打一个标签
      - last_http
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:       #提供watch功能，这里当apache配置文件改变时，就调用handlers中名为"restart apache"的task来重启apache
    - restart apache
  - name: ensure apache is running
    service: name=httpd state=started
  handlers:       #notify通知这里的task执行，谨记：定义在handlers下的task只有在notify触发的时候才会执行
    - name: restart apache
      service: name=httpd state=restarted


执行该playbook文件：

$ ansible-playbook test.yml







⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵 Ansible 进阶 🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️

🔵 Playbook

    其实这个是 状态管理组件! 
    目的是让你用一种更简单的语言 (而不是脚本!) 来描述你的服务应该是怎样的.
    但是 Playbook 不是万能的! 因为 playbook 底层使用各种模块来帮你完成任务.
    虽然现在模块非常多, 但是偶尔还是要写 shell 脚本的.


🔶 基本概念

    ⦿ yaml: 一种文本格式, 你只要知道是用 缩进来表示层级就可以了!!!
    
    ⦿ task: 一个任务!一般用一次模块 就是一个任务
    
    ⦿ play: 一组任务! 从上到下 按顺序执行!!!!

    ⦿ host: 这个playbook 的应用范围.

    ⦿ user: 用什么用户运行 playbook 

    ⦿ role: 角色,一组 playbook, 以及和其配合的元素(vars,files等) 的集合
    
    ⦿ hadnler: 
        task 可以触发一定的事件(比如重启服务), 处理该事件的task就是 handler
        就像我要用ansible 脚本 自动搭建一个 Nginx, 脚本里很多task 都需要重启Nginx. 
        这些 task 都会触发重启Nginx事件,但是其实整个脚本只要重启一次就可以了!
        这时候就可以把 重启Nginx 写到 Handler 里面去! 
        如果有task 触发了 这个重启事件, 不管你触发多少次, 最早只重启一次!


🔵 Roles 

    Role 是非常重要的一个概念! 主要是为了提高代码复用.
    很多时候你只要在自己的 playbook 里引用下别人的 role 即可.
    大家写的 role 可以相互共享.







🔵 facts ✔︎

    默认每次执行 Playbook 前都会自动获取节点设备信息! 比如IP/主机名.
    查看facts: ansible hostname -m setup
    所有这些获取到的信息都可以直接作为变量 应用到 playbook 中.
    这些信息在你用模板的时候比较有用, 
    如果你用不到这些数据, 关闭这个功能, 能非常明显的加快脚本执行速度.
    关闭获取 facts 很简单，只需要在 playbook 文件中加上“gather_facts: no”即可.

        --- 
        - hosts: 172.16.64.240 
        gather_facts: no 





🔵 register  ??

    把任务的输出 保存到某个变量中, 然后用于其他任务.
    这功能对自定义脚本的非常有用! 脚本执行完后肯定有输出来的.
    至于 register 的具体值是什么, 要看task的.
    如果task 是shell. 那么会包含stdout、stderr  还有个 stat 
    要输出 register的值.. 得用  debug...

    🔶 例1

    tasks:

        - name: echo date 
        command: date 
        register: date_output 

        - name: show results?
        debug: msg="{{date_output.stdout}}"

        然后执行 yml 脚本的时候 就会显示一个
        TASK [show results?] ***********************************************************
        ok: [104.224.139.45] => {
            "msg": "Wed Nov  1 19:08:38 CST 2017"
        }

    🔶 例2
    
    - command: "echo {{item}}"
      register: result
      with_items: [1, 2]
    - debug:
        var: result

    这样应该能显示 result 这个值的详细信息!









🔵 Debug 




🔵 命令行变量

    在执行命令的时候传入参数值, 是非常必须的...比如数据库的密码.
    你总不能把密码写在配置文件中吧....

    🔶 定义变量

        - hosts: '{{ hosts }}'
        remote_user: '{{ user }}'
        ...

    🔶 使用变量

        ansible-playbook command.yml --extra-vars "hosts=web user=root"
        



🔵 条件控制

    可用于task，role和include，在满足条件时task才会被执行。
    至于when指令后跟的逻辑表达式也是标准的逻辑表达式，示例如下：        

    tasks:
    - shell: echo "only on Red Hat 6, derivatives, and later"
        when: ansible_os_family == "RedHat" and ansible_lsb.major_release|int >= 6
    - shell: echo "This certainly is epic!"
        when: epic is defined


🔵 循环  

🔶 标准遍历 with_items

    用with_items可以遍历一个列表，注意这里只会遍历一层。示例如下：

    - name: add several users
    user: name={{ item }} state=present groups=wheel
    with_items:
        - testuser1
        - testuser2
            


🔶 嵌套遍历 with_nested

    用with_nested可以遍历一个列表，注意这里会遍历多层，直到最内层...


🔶 配合register循环列表 ??

    register注册一个变量后，可以配合with_items来遍历变量结果。

    示例如下：

    - shell: echo "{{ item }}"
    with_items:
        - one
        - two
    register: echo
    - name: Fail if return code is not 0
    fail:
        msg: "The command ({{ item.cmd }}) did not have a 0 return code"
    when: item.rc != 0
    with_items: echo.results
    