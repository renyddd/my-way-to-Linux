# Ansible
自动化运维神器！[原文参考连接-Ansible 中文权威指南](https://ansible-tran.readthedocs.io/en/latest/docs/playbooks.html)。

ansible 中命令执行的幂等性体现在执行完后的颜色变化中。

通过 ssh 只有纳入到 /etc/ansible/hosts 中的主机才能链接，例如：

 ```bash
# sudo vim /etc/ansible/hosts
192.168.122.91
192.168.122.155
 ```

若此时根据帮助信息直接执行，会有权限报错信息：

```bash
    # usage: ansible [options] HOST-PATTERN
ansible all -m ping
```

### 配置主机 ssh 免密访问

通过证书签名来达到 ssh 的免密访问，`ssh-keygen`与`ssh-copy-id`来实现证书的生成与公钥下发。

首先生成密钥对：

```bash
ssh-keygen -t rsa
```

当前用户家目录下的`.ssh`目录中以`.pub`结尾的便是公钥，接下来通过`ssh-copy-id`进行公钥传输：

```bash
ssh-copy-id -i ./.ssh/id_rsa.pub root@192.168.122.91
```

再无需密码登陆目标主机后，其家目录下的`.ssh`目录下多了两个文件：

- authorized_keys ：存放其他主机的公钥信息；

- known_hosts：

公钥传输完成后 ansible 会试图用你**当前的用户名**来链接目标主机，你可以使用`-u`来指定登陆用户或者`--sudo`模式。

### 常用模块

获取模块（module）列表：

```bash
ansible-doc -l
```

具体模块的使用帮助：

```bash
ansible-doc -s MODULE
```

ansible 模块的更近语法：

```bash
ansible HOST-PATERN -m MOD_NAME -a MOD_ARGS -C -f FORKS

# 
ansible all -m command -a "ls /" -u root

# 文件传输
ansible all -m copy -a "src=/home/renyddd/todolist.md dest=/todolist.md" -u root

# 包管理
ansible all -m yum -a "name=tree state=present" -u root

#服务管理
ansible all -m service -a "name=network state=started" -u root
```



## Playbooks

Playbooks 是 Ansible 的配置、部署、编排语言，可以被描述为是一个希望远程主机执行命令的方案。

Playbooks 的格式是`YAML`:

### YAML 语法

Ansible 中的每一个 YAML 文件都是从一个列表开始，而且开始行都应该由`---`，这是 YAML 的一部分表明文件的开始。

列表中的所有成员都开始与相同的缩进级别，并且以`- `（横杠加空格）作为开头。

字典是由一个`key: value`键值对组成。

并且在 Ansible 当中使用`"{{ varname}}"`的形式来引用变量。

```yam
---
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
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
```

### 简介

Playbook 由一个或多个的 ‘plays’ 组成，他的内容是一个以 ‘plays’ 为元素的列表。而在 play 之中一组机器被映射为定义好的角色，play 的内容被称为 task 即为任务。一个任务就是一个对 ansible 模块的调用。

前面的示例即为仅包含了一个 play 的 playbook。

#### Hosts 行

你可以分别为每一个 play 指定目标主机与身份用户，因此 hosts 行的内容即为一个或多足主机的 pattern。

```yaml
---
- hosts: dbserver
  remote_user: root
  tasks:
  - name: test connection
    ping:
    remote_user: yourname
```

 也可以向上面一样为每一个 task 单独指定远程用户。

#### Tasks 列表

一个 task 在其所有主机上执行完毕之后才会进行下一个 task，每个 task 目标在于执行一个 module，由于幂等性的存在所以多次重复也很安全。

每个 task 必须要有一个 name，其后的格式为`module: options `：

```yaml
---
- hosts: all
  remote_user: root
  tasks:
      - name: test connection
        ping:
```

#### Handlers

Playbook 可以识别到远端系统的改动，`notify`会在每一个 task 结束时被触发，即使有多个不同的 task 通知改动的发生`notify`也只会被触发一次。

待补充。 

#### 执行 playbook

```bash
ansible-playbook testBook.yml
```

### Roles 和 include

为提高 playbook 的重用性，借助`include`语句引用 task 文件可以帮你将整个策略分到更多的小文件当中。















