## 文件管理，部署和配置管理
### 管理文件
#### 复制本地文件
#### SCP：安全的复制
#### Rsync：在不同主机间同步文件

Rsync 是另一种安全传输文件的方式。它可以检测到文件变化，通过仅仅同步那些变化的文件来节省带宽。
类似于SCP，rsync也是利用ssh来连接远程主机，向它们发送或从它们那接收文件。大部分情况下，类似的规则和ssh相关的选项也适用于rsync。

```sh
rsync /path/to/source/file.ext username@hostname.com:/path/to/destnation/file.ext
```

##### 有用的选项
-r: 用于复制文件夹，使用recursive（递归）标记
-e: 用于选择/改变ssh和它的选择设置，如`-e "ssh -p 8888 -i ~/.ssh/xxx_identity.pem"`
-z: 压缩文件
-c: 根据checksum而不是修改时间来对比文件
-S: 有效的处理稀疏文件
-p: 保留文件权限相关
-h: 以友好可读的方式显示数字
--exclude=""：用于排除特定文件夹的同步。如 `--exclude=".git"`

##### 同步前先预览
##### 恢复之前上传
##### 打包选项
##### 智能的文件夹间合并

#### 部署
### 用 Github 自动部署


### 用 Ansible 管理配置
Ansible 是个配置管理和提供的工具，类似于 Chef, Puppet 或 Salt。
我发现它上手起来是最简单易用的。大部分是因为它仅仅就是 SSH，它利用 SSh 连接服务器和允许配置好的任务。
使用Ansible有点很棒的特性就是它可以非常容易的把 bash 脚本（仍然是非常流行的方式来实现和运行配置管理方式）转换为 Anisble 任务。既然它主要是基于 SSH 的，那我们不难发现它为什么可以方便的实现这样的特性 - 因为它就是在那些机器上运行这些脚本。
我们只需要提供好我们的独立脚本，Ansible让我们脚本很清晰因为它让运行前的获得上下文的处理自动化了。利用这上下文，Ansible能够处理绝大部分的边缘 case - 那些我们之前通常需要编写又长又臭的复杂脚本来处理。
Ansible的任务是冥等的，意味着我们可以跑同样的系列任务一遍又一遍不需要考虑带来的负面影响。如果不通过不少编码工作，普通的bash脚本是很难有这样的确保的。
为了实现这冥等性，Ansible使用『facts』这个概念，它就是在运行任务之前收集的关于系统和环境的一些信息。这些事实信息用来检查系统的状态，决定是否需要执行些操作以得到预期的结果。
接下来我会展示上手Ansible是件多么容易的一件事。先通过基础级别的任务和添加更多特性功能的展示，我们从而不断改善着 Ansible 配置。

#### 安装

使用前必须安装，这肯定是必须的。
这就是意味着通常我们需要一台中心服务器来运行Ansible的命令，但 Ansible 对所安装的服务器没有什么特殊的要求 - 我通常在我自己的笔记本上运行这些Ansible任务。要知道，Ansible是agentless（无需代理人）的。
接下来我们展示如何在Ubuntu14.04上安装我们的Ansible。利用官方提供的非常好记的ppa源： `ppa:ansible/ansible`项目

```sh
sudo apt-add-repository -y ppa:ansible/ansible
sudo apt-get update
sudo apt-get install -y ansible
```

#### 管理服务器

Ansible有个默认的库存配置文件来来定义它要管理哪些服务器。安装完后，在/etc/ansible/hosts下通常有个官方提供的这样文件。

`sudo mv /etc/ansible/hosts /etc/ansible/hosts.orig`

然后我一般会从新创建一个我们自己的库存配置文件。譬如下面我们定义两台使用web标签的服务器。
如果有需要，我们还可以定义一系列的主机，多个组，可重用的变量，使用其他配置玩法，包括创建动态的机器库存配置。
接下来，为了方便演示，我创建台虚拟机，安装Ansible，然后在这台服务器上运行Ansible任务。配置很简单，如下local标签所记：
这样测试变得很容易，我们不需要真的有多台服务器或多个虚拟机。接下来我要告诉ansible使用 vagrant 用户和密码来执行任务而不是密钥授权。

`PS: 需要注意的是，这不是典型做法。只是为了便于展示`

```sh
[web]
192.168.22.10
192.168.22.11

[local]
127.0.0.1

```

#### 基础：运行命令

ansible all -m ping
ansible all -m ping -s -k -u vagrant

all：使用库存配置文件中所有定义的服务器来执行任务
-m ping : 使用ping模块，它简单的运行ping命令并且返回运行结果
-s：使用sudo来执行命令
-k：使用密码而不是密钥认证鉴权
-u vagrant：用vagrant用户登录服务器

##### Modules模块

ansible 使用模块来完成大部分任务。模块可以实现安装软件，复制文件，使用模板等等。
模块可以结合着现有的上下文（facts）来决定我们是否要执行哪些任务，来实现我们的预期效果。
如果我们不用模块，我们也可以仅仅运行任意的shell命令。如`ansible all -s -m shell -a 'apt-get install nginx'`

#### 基础的 Playbook

Playbooks能够运行多个任务，提供一些仅仅通过执行shll命令没有的更高级的功能。

`PS: 在Ansible中，Playbooks和角色都是通过Yaml格式配置`

创建如下的nginx.yml

```yaml
---
- hosts: local
  tasks:
    - name: Install Nginx
      apt: pkg=nginx state=installed update_cache=true
```

`ansible-playbook -s nginx.yml`

##### 处理者Handlers
处理者和任务很类似（它能够完成任务能完成的一切），但它可以被另外的任务调用执行。你可以把它当成事件系统的一部分。当它监听的事件发生了，处理者会开始执行。
一些情况下次级行为对运行任务来说是很必须的。如在安装完服务后启动它，或在配置改变后重新加载它。

```yaml
---
- hosts: local
  sudo: yes
  tasks:
    - name: Install Nginx
      apt: pkg=nginx state=installed update_cache=true
      notify:
        - Start Nginx

handlers:
 - name: Start Nginx
   service: name=nginx state=started
```

##### 更多任务

接下来我们给Playbook添加更多的任务，看看其他一些功能。

现在有三个任务了：

- 添加Nginx仓库
- 安装Nginx：使用apt模块安装
- 创建 Web 根目录

配置中我们也用了两个新的指令：register（注册）和when（何时）。这些告诉Ansible在一些事情发生时运行些任务。
Add Nginx Repository（添加Nginx仓库）这个任务我们注册了 ppastable。接着我们用它通知安装Nginx这个任务仅仅当注册的ppastable任务被成功执行后才开始运行。这运行我们来条件化让Ansible停止运行任务。
我们也使用了一个变量，在var板块中定义的 docroot 变量。它被文件模块用做dest目的地参数来创建预期的目录。
接下来，我们会通过把Playbook组织入Role角色，来进一步探索Ansible的更多强大功能。

#### 角色
角色非常适合组织多个相关的任务，来封装需要完成这些任务的数据。例如安装Nginx可能涉及到添加包仓库，安装包，确定配置文件等。在之前的Playbook我们看到如何安装，不过一旦开始配置我们的安装，Playbook就开始有点吃力了。
配置部分通常需要一些诸如变量，文件，动态模板等额外数据。这些东西可以在Playbook中被使用，但是通过组织相关的任务和数据到一个统一的结构体：Role角色中，我们可以获得更好的可读性。
角色一般有如下的目录结构：

```sh
rolename
    /files
    /handlers
    /meta
    /templates
    /tasks
    /vars
```

在每个目录下，Ansible会自动寻找和读取那些命名为main.yml的配置文件。我们把之前的nginx.yml拆分到各个组成部分到对应的目录中，从而得到了更清晰和完整的配置提供链。

#### 事实
#### Vault

