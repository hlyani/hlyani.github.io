

# ansible 相关

# 一、基础使用

## 1、配置文件

- ansible 应用程序的 主配置文件：/etc/ansible/ansible.cfg

- Host Inventory 定义管控主机 ：/etc/ansible/hosts

  ```hosts
  [both]
  master
  node1
  node2
  
  [node]
  node1
  node2
  ```

遵循 INI风格；中括号中的字符是组名；一个主机可同时属于多个组；

## 2、ansible-doc命令：获取模块列表，及模块使用格式；

```
ansible-doc -l ：获取列表
ansible-doc -s  module_name ：获取指定模块的使用信息
```

## 3、模块

- command 默认模块，可省略
- shell 以shell解释器执行脚本
- raw
- script 将本地脚本传送到远端节点上执行

## 4、变量使用示例：

Tasks 任务、 Variables 变量、 Templates 模板、 Handlers 处理器、 Roles 角色

```
[root@localhost~]# vim useradd.yml
-hosts: websrvs
    remote_user: root
    vars:
    username: testuser
    password: xuding
    tasks:
-name: add user
    user: name={{ username }} state=present
    -name: set password
    shell: /bin/echo {{ password }} |/usr/bin/passwd --stdin {{ username }}
```

- 调用变量`\{\{ \}\}`

- ansible-playbook /PATH/TO/SOME_YAML_FILE  { -eVARS|--extra-vars=VARS}  变量的重新赋值调用方法

```
[root@localhost ~]# ansible-playbook useradd.yml --extra-vars "username=ubuntu"
playbook--- tasks
```

## 5、条件测试：

在某task后面添加when子句即可实现条件测试功能；when语句支持Jinja2语法；

实例：当时 RedHat 系列系统时候调用 yum 安装

```
tasks:
    -name: install web server package
    yum: name=httpd state=present
    when: ansible_os_family == "RedHat"
```

## 6、迭代： item

在task中调用内置的item变量；在某task后面使用with_items语句来定义元素列表；

```
tasks:
    -name: add four users
with_items:
    -testuser1
    -testuser2
    -testuser3
    -testuser4
```

注意：迭代中，列表中的每个元素可以为字典格式；

实例：

- ```
  -name: add two users
  user: name={{ item.name }}  state=present groups={{ item.groups }}
  with_items:
      - { name: 'testuser5', groups: 'wheel' }
      - { name: 'testuser6', groups: 'root' }
  ```


playbook--- handlers： 处理器；触发器

只有其关注的条件满足时，才会被触发执行的任务；

## 7、playbook 模板

- templates：

  用于生成文本文件（配置文件）；模板文件中可使用jinja2表达式，表达式要定义在`\{\{\}\}`，也可以简单地仅执行变量替换；

- roles：

  roles用于实现“代码复用”；

  roles以特定的层次型格式组织起来的playbook元素（variables,tasks, templates, handlers）；

  可被playbook以role的名字直接进行调用；

  用法 ：在 roles/ 下建立 [group_name] 子目录，并非全部都要创建；例如：

  /etc/ansible/roles/ （在 /etc/ansible/ansible.cfg 定义 roles 目录）

  webserver/

  files/：此角色中用到的所有文件均放置于此目录中；

  templates/：Jinja2模板文件存放位置；

  tasks/：任务列表文件；可以有多个，但至少有一个叫做main.yml的文件；

  handlers/：处理器列表文件；可以有多个，但至少有一个叫做main.yml的文件；

  vars/：变量字典文件；可以有多个，但至少有一个叫做main.yml的文件；

  meta/：此角色的特殊设定及依赖关系；

  ```
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

# 二、常用命令

```
# 拷贝文件
ansible all -m copy -a "src=/etc/profile dest=/etc force=yes"

# ping
ansible all -m ping
ansible  all  -m  ping  -u  root  --ask-pass

# 启动服务
ansible webservs -m service -a 'enabled=true name=httpd state=started'

# 管道符
ansible all -m shell -a 'echo 123..com | passwd --stdin user1'

# 执行脚本
ansible all -m script -a "/tmp/a.sh"

# 安装程序，卸载加state=absent
ansible all -m yum -a "name=zsh"
ansible all -m yum -a "name=zsh state=absent"

# 使用git部署webapp
ansible webservers -m git -a "repo=git://foo.example.org/repo.git dest=/srv/myapp version=HEAD"

# touch 一个文件并添加用户读写权限，用户组去除写执行权限，其他组减去读写执行权限
ansible web -m file -a  "path=/etc/foo.conf state=touch mode='u+rw,g-wx,o-rwx'"

# 使用file 模块创建目录，类似mkdir -p
ansible web -m file -a "dest=/tmp/test mode=755 owner=user group=user state=directory"

# 使用file 模块删除文件或者目录
ansible web -m file -a "dest=/tmp/test state=absent"

# 搜集主机的所有系统信息
ansible all -m setup

# 搜集和内存相关的信息
ansible all -m setup -a 'filter=ansible_*_mb'

# 搜集网卡信息
ansible all -m setup -a 'filter=ansible_eth[0-2]'

# 搜集系统信息并以主机名为文件名分别保存在/tmp/facts 目录
ansible all -m setup --tree /tmp/facts
```

# 三、其他

## 1、创建目录

```
- name: 创建目录
  file: name={{ tmp_dir }} state=directory
  connection: local
  run_once: true
```

## 2、执行 shell

```
- name: shell
  shell:
    cmd: |
      \cp -rf "{{ base_dir }}/tmp" "{{ tmp_dir }}"
  connection: local
  run_once: true
```

## 3、渲染 value.yml.j2

```
- name: render value.yaml
  template: src={{ item }}.j2 dest={{ tmp_dir }}/{{ item }}
  with_items:
    - values.yaml
  connection: local
  run_once: true
```

## 4、循环 group

```
- name: loop group
  shell: echo {{ item }}
  with_items: "{{ groups['tmp'] }}"
  connection: local
  run_once: true
```

```
vim label-config.yml
labels:
  a: b
  c: d
```

```
- include_vars: "{{ cluster_dir }}/label-config.yml"
- name: label
  shell:
    cmd: |
      hosts=`echo "{{ item.value }}"|tr -d "',[]"`
      label="{{ labels[item.key] }}"
      for host in ${hosts};do
        echo $host $label
      done
  when: item.key in labels
  loop: "{{ groups|combine|dict2items }}"
```

> list -> dict -> items

## 5、变量渲染

[https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html#playbooks-filters](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html#playbooks-filters)

```
{{ affinity | to_yaml | indent(2) }}
```

```
{{ nodeSelector | to_nice_yaml }}
```

```
{{ nginx.configurationSnippet.httpStart | to_nice_yaml | from_yaml | indent(8) }}
```

## 6、引用变量

```
ansible-playbook -i clusters/test/hosts -e @clusters/test/config.yml -e @clusters/test/lable-config.yml playbooks/106.test.yaml -vvv
```

```
- hosts: localhost
  tasks:
    - include_vars: "{{ cluster_dir }}/label-config.yml"
```

## 7、过滤所有注释行

```
grep -v '^[[:blank:]]*#' values.yaml.j2
```

## 8、重试循环

```
- action: shell /usr/bin/foo
  register: result
  until: result.stdout.find("all systems go") != -1
  retries: 5
  delay: 10
```

