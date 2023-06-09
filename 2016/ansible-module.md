ansible
<!--more-->
## 运维自动化工具介绍
> 在日常服务器维护中，从系统安装到程序部署再到发布应用，在大规模的生产环境中，如果需要手动的每台服务器进行安装配置将会给运维人员带来许多繁琐而又重复的工作。这就促使了在每个运维层次中出现了不同的自动化运维工具。
常见的自动化运维工具分类有以下几类：
 
### 系统安装运维工具（OS Provisioning）：
 > 常见的有：PXE,Cobbler，Red Hat Satelite(redhat)系统专用等

 ### 操作系统的配置运维工具(OS Config)：
 > 常见的有：cfengine，puppet,saltsack,chef等 

 ### 应用程序部署工具(Application Service Orchestration):
 > 常见的有:Func,Fabric,ControITier,Capistrano等

### 根据工作模式不同上面的运维工具有分为以下两类：
  * agent：基于ssl协议实现，agent工作在被监控端，例如：puppet
  * agentless: 基于ssh key实现，例如：ansible

## ansible介绍
>  ansible是一款轻量级自动化运维工具，由Python语言开发，结合了多种自动化运维工具的特性，实现了批量系统配置、批量程序部署、批量命令执行等功能；ansible是基于模块化实现批量操作的。

![](http://upload-images.jianshu.io/upload_images/1542757-c4b2d6eede79a975.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## ansible组成
* Ansible：            核心
* Modules：          包括 Ansible 自带的核心模块及自定义模块
* Plugins：            完成模块功能的补充，包括连接插件、邮件插件等
* Playbooks：       网上很多翻译为剧本，个人觉得理解为编排更为合理；定义 Ansible 多任务配置文件，有 Ansible 自动执行
* Inventory：         定义 Ansible 管理主机的清单

## ansible特点
>模块化、部署简单、工作于agentless模式、默认使用ssh协议、支持自定义模块、支持Palybook等	

## ansible 基本安装介绍
### 系统环境

```python
[root@note1 ~]# uname -a
Linux note1 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@note1 ~]# cat /etc/redhat-release 
CentOS release 6.6 (Final)
```
### epel源 
```python
 wget -O /etc/yum.repos.d/epel.repo  http://mirrors.aliyun.com/repo/epel-6.repo
```  
### 安装ansible
```python
[root@note1 ~]# yum -y install python-jinja2 PyYAML python-paramiko python-babel python-crypto ansible
```
### 配置ansible主机文件 
```python
[root@note1 ~]# > /etc/ansible/hosts 		 #清空文件(操作前先备份)
[root@note1 ~]# vim /etc/ansible/hosts  	  #编辑配置主机文件
[root@note1 ~]# cat /etc/ansible/hosts        #查看配置
[web]
192.168.70.51
[db]
192.168.70.50
```
### 配置主机免密钥登陆
> 为了避免Ansible下发指令时输入目标主机密码，通过证书签名达到SSH无密码是一个好的方案，推荐使用ssh-keygen与ssh- copy-id来实现快速证书的生成和公钥下发，其中ssh-keygen生成一对密钥，使用ssh-copy-id来下发生成的公钥。具体操作如下：

```python
[root@note1 ~]ssh-keygen -t rsa -P ''
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
a8:eb:cc:da:26:06:67:9b:23:ae:45:04:d4:76:63:bb root@bj-idc-15
The key's randomart image is:
+--[ RSA 2048]----+
|o..              |
| . o +           |
|  o o o          |
| .   . .         |
|  .   o S        |
|..o  E           |
| +.o.            |
|..*+..           |
|++.**            |
```
### 密钥拷贝到远程主机
```python
[root@note1 ~]# ssh-copy-id  -i ~/.ssh/id_rsa.pub root@192.168.70.51
root@192.168.70.51's password: 
Now try logging into the machine, with "ssh 'root@10.10.10.14'", and check in:
  .ssh/authorized_keys
to make sure we haven't added extra keys that you weren't expecting.

[root@note1 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.70.50
root@192.168.70.50's password: 
Now try logging into the machine, with "ssh 'root@192.168.70.50'", and check in:
  .ssh/authorized_keys
to make sure we haven't added extra keys that you weren't expecting.

*注意：在首次连接或者重装系统之后会出现检查 keys 的提示*
```
### 测试ping
```python
[root@note1 ~]# ansible all -m ping 
The authenticity of host '192.168.70.50 (192.168.70.50)' can't be established.
RSA key fingerprint is 56:49:f5:4f:3e:30:8d:5c:2c:42:06:79:69:a4:18:89.
Are you sure you want to continue connecting (yes/no)?
解决办法:
vim /etc/ansible/ansible.cfg 或者 ~/.ansible.cfg
[defaults]
host_key_checking = False
也可以通过设置系统环境变量来禁止这样的提示
export ANSIBLE_HOST_KEY_CHECKING=False
应用最后一个办法
[root@note1 ~]# export ANSIBLE_HOST_KEY_CHECKING=False
再次测试
[root@note1 ~]# ansible all -m ping                                
192.168.70.50 | success >> {
    "changed": false, 
    "ping": "pong"
}
192.168.70.51 | success >> {
    "changed": false, 
    "ping": "pong"
}
```
### Ansible命令参数介绍
```python
-v,–verbose   详细模式，如果命令执行成功，输出详细的结果(-vv –vvv -vvvv)
-i PATH,–inventory=PATH   指定host文件的路径，默认是在/etc/ansible/hosts 
-f NUM,–forks=NU  NUM是指定一个整数，默认是5，指定fork开启同步进程的个数。 
-m NAME,–module-name=NAME   指定使用的module名称，默认是command
-m DIRECTORY,–module-path=DIRECTORY   指定module的目录来加载module，默认是/usr/share/ansible, 
-a,MODULE_ARGS   指定module模块的参数 
-k,-ask-pass     提示输入ssh的密码，而不是使用基于ssh的密钥认证
-sudo                   指定使用sudo获得root权限
-K,-ask-sudo-pass       提示输入sudo密码，与–sudo一起使用 
-u USERNAME,-user=USERNAME  指定移动端的执行用户 
-C,-check               测试此命令执行会改变什么内容，不会真正的去执行

```
###主机清单介绍hosts
> Ansible 通过读取默认的主机清单配置/etc/ansible/hosts，可以同时连接到多个远程主机上执行任务，默认路径可以通过修改 ansible.cfg 的 hostfile 参数指定路径。

```python
[dbserver]  []表示主机的分组名,可以按照功能,系统进行分类,便于进行操作
192.168.10.2 
one.example.com 
www.bds.com:5309         #支持指定ssh端口5309 
jumper ansible_ssh_port=5309 ansible_ssh_host=192.168.10.2   #设置主机别名为jumper
www[01:50].bds.com       #支持通配符匹配www01.bds.com www02.bds.com
[web]				    #提醒下这里面字母是随便定义的
web-[a:f].bds.com        #支持字母匹配 web-a.bds.com ..web-f.bds.com
```
### 为主机指定类型和连接用户
```python
[bds]
Localhost  ansible_connection=local
other1.example.com ansible_connection=ssh ansible_ssh_user=deploy
other2.example.com ansible_connection=ssh ansible_ssh_user=deploy
ansible hosts配置文件中支持指令
```

**注意: 前面如果不配置主机免密钥登录,可以在/etc/ansible/hosts中定义用户和密码,主机ip地址,和ssh端口,这样也可以进行免密码访问,但是这个/hosts文件要保护好,因为所有的密码都写在里面**
### hosts文件配置参数介绍
---
>1, ansible_ssh_host :                     
指定主机别名对应的真实 IP，如：100 ansible_ssh_host=192.168.1.100，随后连接该主机无须指定完整 IP，只需指定 251 就行
2, ansible_ssh_port :                         
指定连接到这个主机的 ssh 端口，默认 22
3, ansible_ssh_user:                          
连接到该主机的 ssh 用户
4, ansible_ssh_pass:                           
连接到该主机的 ssh 密码（连-k 选项都省了），安全考虑还是建议使用私钥或在命令行指定-k 选项输入
5, ansible_sudo_pass:                          	sudo 密码
6, ansible_sudo_exe:                            	sudo 命令路径
7, ansible_connection :        
连接类型，可以是 local、ssh 或 paramiko，ansible1.2 之前默认为 paramiko
8, ansible_ssh_private_key_file :               私钥文件路径
9, ansible_shell_type  :                       
目标系统的 shell 类型，默认为 sh,如果设置 csh/fish，那么命令需要遵循它们语法
10, ansible_python_interpreter  :      
python 解释器路径，默认是/usr/bin/python，但是如要要连*BSD系统的话，就需要该指令修改 python 路径
11, ansible_*_interpreter  :    
这里的"*"可以是 ruby 或 perl 或其他语言的解释器，作用和 ansible_python_interpreter 类似



## ansible 常用模块介绍
### ansible使用帮助
```python
[root@note1 ~]# ansible-doc  -l                 #查询ansible的所有模块
[root@note1 ~]# ansible-doc -s module_name  	#查看模块的属性信息
```
### 例子: 查询service 模块的信息 
```python
[root@note1 ~]# ansible-doc -s service		   
less 436
Copyright (C) 1984-2009 Mark Nudelman

less comes with NO WARRANTY, to the extent permitted by law.
For information about the terms of redistribution,
see the file named README in the less distribution.
Homepage: http://www.greenwoodsoftware.com/less
- name: M a n a g e   s e r v i c e s .
  action: service
      arguments              # Additional arguments provided on the command line
      enabled                # Whether the service should start on boot. *At least one of state and enabled are requi
      name=                  # Name of the service.
      pattern                # If the service does not respond to the status command, name a substring to look for as
      runlevel               # For OpenRC init scripts (ex: Gentoo) only.  The runlevel that this service belongs to.
      sleep                  # If the service is being `restarted' then sleep this many seconds between the stop and 
      state                  # `started'/`stopped' are idempotent actions that will not run commands unless necessary
(END)
```

### ansible语法
> ansible <pattern_goes_here> -m <module_name> -a <arguments>

### command模块 
> 默认模块 ,用于在各个被管理节点运行指定的命令(不支持管道和变量)

```python
[root@note1 ~]# ansible all -m command -a "hostname "
192.168.70.51 | success | rc=0 >>
bds
192.168.70.50 | success | rc=0 >>
note1
```
### shell模块
>command模块功能相同，但比command的模块功能强大(支持管道和变量)

```python
[root@note1 ~]# ansible all -m shell -a "cat /etc/passwd| grep root "                         
192.168.70.51 | success | rc=0 >>
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
192.168.70.50 | success | rc=0 >>
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
```
### user模块
> 用户模块,用于在各管理节点管理用户所使用
创建一个名字为DBA的用户,uid是505 ,家目录是/Data/dba,shell是不让用户登录

```python
[root@note1 ~]# ansible db -m user -a 'name=DBA uid=505 home=/Data/dba shell=/sbin/nologin'            
192.168.70.50 | success >> {
    "changed": true, 
    "comment": "", 
    "createhome": true, 
    "group": 505, 
    "home": "/Data/dba", 
    "name": "DBA", 
    "shell": "/sbin/nologin", 
    "state": "present", 
    "system": false, 
    "uid": 505
}
```
> 删除一个用户

```python
ansible db -m user  -a 'name=budongshu uid=506  state=absent'
```
### group模块
```python
ansible db -m group  -a 'name=test  gid=1000' 
```
### cron模块
>计划定时任务,用于在各管理节点管理计划任务 

```python
[root@note1 ~]# ansible all -m cron -a "name=time minute='*/2' job='/usr/sbin/ntpdate 
 ```
### copy模块
>复制模块,复制文件到各个节点

```python
[root@note1 ~]# ansible all -m copy -a "src=/etc/hosts dest=/tmp/ mode=600"
 ```
### file模块 
> 文件模块 , 修改各个节点指定的文件属性

```python
[root@note1 ~]# ansible all -m file -a 'path=/tmp/hosts mode=644 owner=DBA'  
```

> 创建目录类似mkdir –p

```python
[root@note1 ~]# ansible all -m file -a "dest=/tmp/ansible.txt mode=755 owner=root 
group=root state=directory"
```
> file删除文件或者目录

```python
[root@note1 ~]# ansible all -m file -a "dest=/tmp/ansible.txt state=absent"   
 ```
**注：state的其他选项：link(链接)、hard(硬链接)** 

### stat 模块
> 获取远程文件状态信息，包含atime、ctime、mtime、md5、uid、gid等

```python
[root@note1 ~]# ansible all -m stat -a "path=/etc/passwd "
```
### ping 模块
> 测试模块 ,测试各个节点是否正常在线

```python
 ansible all -m stat -a 'path=/etc/passwd' 
```
### template模块
> 根据官方的翻译是：template使用了Jinjia2格式作为文件模板，进行文档内变量的替换的模块。他的每次使用都会被ansible标记为changed状态。

### yum模块
> 用于管理节点安装软件所使用

```python
[root@note1 ~]# ansible all -m yum -a 'name=ntp state=present'
卸载的软件只需要将 name=ntp state=absent 
安装特定版本 name=nginx-1.6.2 state=present
指定某个源仓库安装软件包name=htop enablerepo=epel state=present
更新软件到最新版 name=nginx state=latest
``` 
### service模块
> 管理各个节点的服务

```python
[root@note1 ~]# ansible all -m service -a "name=ntpd enabled=true state=started"     state 支持其它选项 started stopped restarted 
 ``` 

### script模块
> 自动复制脚本到远程节点,并运行

* 测试脚本
```python
[root@note1 ~]# cat ansible_test.sh 
#!/bin/bash 
echo "hello world " >> /tmp/ansible_sh.log
```
* 运行脚本
```python
[root@note1 ~]# ansible all -m script -a 'ansible_test.sh'
 ```

### setup模块

> 收集ansible的facts信息


```python
[root@note1 ~]# ansible all -m setup  #收集主机的facts信息,可以通过变量引用这些信息
```


### ansible 主机清单通配模式介绍
> 可以看到上面执行命令的时候有个ansible -m all ,以上我用的all或指定主机,这里也可以进行通配 ,在/etc/ansible/hosts 进行设置如下

```python
[web]
10.10.10.2
10.10.10.3
[db]
10.10.10.4 
[allhost:children]     #可以把一个组当做另一个组的子成员
web
db
```
### 例子:
```python
ansible web -m shell -a ‘uptime’     #代表web组中的所有主机
ansible allhost -m shell -a ‘uptime’ #代表allhost组中的所有子成员组
```

###  其它匹配方式

> 1.1 通配所有主机
all , *
ansible all  -m shell -a ‘uptime’
ansible *   -m shell –a ‘uptime’

> 1.2 通配具有规则特征的主机或者主机名
one.bds.com
*.bds.com
192.168.10.2
192.168.10.* 

> 1.3 通配俩组的所有主机,组名之间通过冒号分开,表示or的意思
web:db

> 1.4 非模式匹配: 表示在 web组不在db组的主机
web:!db

> 1.5 交集匹配: 表示同时都在 web 和db组的主机
web:&db

>1.6 匹配一个组的特定编号的主机 从零开始计算
web[0]

>1.7 匹配 web组的第 1 个到第 25 个主机
web [0-25]      

>1.8  组合匹配
在web组或者在db组中,必须还存在test1组中,但不在test2组中
web:db:&test1:!test2  

>1.9 大部分人都在patterns应用正则表达式,但你可以.只需要以 ‘~’ 开头即可:
~(web|db).*\.example\.com

>2.0 同时让我们提前了解一些技能,除了如上,你也可以通过 --limit 标记来添加排除条件,/usr/bin/ansible or /usr/bin/ansible-playbook都支持:
ansible-playbook site.yml --limit datacenter2

>2.1 如果你想从文件读取hosts,文件名以@为前缀即可.从Ansible 1.2开始支持该功能:
ansible-playbook site.yml --limit @retry_hosts.txt
