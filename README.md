# Ansible-DevOps

    自学的人好苦逼啊, 不写个项目出来都不敢说自己会配置管理! 
    配置管理那么多: Puppet、Ansible、Chef....
    Puppet 太重量级了! 适合人家大公司的不一定适合中小公司!!!
    Ansible 应该是中小公司最佳的选择. Chef.. 没啥了解...
    自己这水平也进不了大公司,于是就选择了 Ansible.

    这个项目的目的就是为了可以自动化安装一个SSR !
    你要做的只是 开启一个新的vps, 然后执行一条命令! 


# TODO 





# Done 

    1. Ansible 搭建(服务端+节点)  2017-10-28-22 ✔︎





⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵 Ansible 搭建 🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️

🔵  Ansible 安装 CentOS_7 ✔︎

    yum install epel-release
    yum search ansible 
    yum install ansible -y
    ansible --version

🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶 密钥配置 ✔︎ 🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶

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




🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶 Ansible 节点配置 ✔︎ 🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶

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

🔶 AWS 节点再次测试 ✔︎

    ✘✘∙GCE ansible ➜ ansible -m ping 'aws'
    13.115.230.116 | SUCCESS => {
        "changed": false,
        "failed": false,
        "ping": "pong"
    }    





⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️
🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵 Ansible 基础知识 🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️------⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️⬛️

🔶 简介

    只要 Ansible 能成功连上客户端, 那么就算搭建成功了! 非常简单的!
    当然会搭建 不等于说你会 Ansible 了...  连皮毛都还没学会呢..继续...
    通过上面的搭建,对Ansible 还是没啥了解! 首先我们先学下 Ansible 基础! 


🔶 Ansible 













🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶  软件安装  🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶🔶






