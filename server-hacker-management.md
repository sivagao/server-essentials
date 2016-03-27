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
#### 安装
#### 管理服务器
#### 基础：运行命令
#### 基础的 Playbook
#### 角色
#### 事实
#### Vault

