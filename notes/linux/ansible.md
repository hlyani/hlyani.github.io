

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
- ping
- yum
- template
- copy
- user
- group 模块用于在受控机上添加或删除组
- service
- script

常用文件模块

| 模块名称    | 模块说明                                                     |
| ----------- | ------------------------------------------------------------ |
| blockinfile | 插入、更新或删除由可自定义标记线包围的多行文本块             |
| copy        | 将文件从本地或远程计算机复制到受管主机上的某个位置。 类似于file模块，copy模块还可以设置文件属性，包括SELinux上下文件。 |
| fetch       | 此模块的作用和copy模块类似，但以相反方式工作。此模块用于从远程计算机获取文件到控制节点， 并将它们存储在按主机名组织的文件树中。 |
| file        | 设置权限、所有权、SELinux上下文以及常规文件、符号链接、硬链接和目录的时间戳等属性。 此模块还可以创建或删除常规文件、符号链接、硬链接和目录。其他多个与文件相关的 模块支持与file模块相同的属性设置选项，包括copy模块。 |
| lineinfile  | 确保特定行位于某文件中，或使用反向引用正则表达式来替换现有行。 此模块主要在用户想要更改文件的某一行时使用。 |
| stat        | 检索文件的状态信息，类似于Linux中的stat命令。                |
| synchronize | 围绕rsync命令的一个打包程序，可加快和简化常见任务。 synchronize模块无法提供对rsync命令的完整功能的访问权限，但确实最常见的调用更容易实施。 用户可能仍需通过run command模块直接调用rsync命令。 |

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

## 8、debug

| 选项  | 描述                                                         |
| ----- | ------------------------------------------------------------ |
| -v    | 显示任务结果                                                 |
| -vv   | 任务结果和任务配置都会显示                                   |
| -vvv  | 包含关于与受管主机连接的信息                                 |
| -vvvv | 增加了连接插件相关的额外详细程序选项，包括受管主机上用于执行脚本的用户以及所执行的脚本 |

## 9、语法验证

```
ansible-playbook --syntax-check webserver.yml
```

## 10、执行空运行

> -C选项对playbook执行空运行。使Ansible报告在执行该playbook时将会发生什么更改，但不会对受管主机进行任何实际的更改。

```
ansible-playbook -C webserver.yml 
```

## 11、实施处理程序

```
tasks:
  - name: copy demo.example.conf configuratioon template      # 通知处理程序的任务
    template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:         # notify语句指出该任务需要触发一个处理程序
      - restart apache     # 要运行的处理程序的名称

handlers:       # handlers关键字表示处理程序任务列表的开头
  - name: restart apache   # 被任务调用的处理程序的名称
    service:    # 用于该处理程序的模块
      name: httpd
      state: restarted
```

## 12、ansible 角色子目录

| 子目录        | 功能                                                         |
| ------------- | ------------------------------------------------------------ |
| **defaults**  | 此目录中的**main.yml**文件包含角色变量的默认值，使用角色时可以覆盖这些默认值。 这些变量的优先级较低，应该在play中更改和自定义。 |
| **files**     | 此目录包含由角色任务引用的静态文件。                         |
| **handlers**  | 此目录中的**main.yml**文件包含角色的处理程序定义。           |
| **meta**      | 此目录中的**main.yml**文件包含与角色相关的信息，如作者、许可证、平台和可选的角色依赖项。 |
| **tasks**     | 此目录中的**main.yml**文件包含角色的任务定义。               |
| **templates** | 此目录包含由角色任务引用的Jinja2模板。                       |
| **tests**     | 此目录可以包含清单和名为**test.yml**的playbook，可用于测试角色。 |
| **vars**      | 此目录中的**main.yml**文件定义角色的变量值。这些变量通常用于角色内部用途。 这些变量的优先级较高，在playbook中使用时不应更改。 |

## 13、导入 task

task.yaml

```
---
- name: Install the {{ package }} package
  yum:
    name: "{{ package }}"
    state: latest
- name: Start the {{ service }} service
  service:
    name: "{{ service }}"
    enabled: True
    state: started
```

```
  tasks:
    - name: Import task file and set variables
      import_tasks: task.yml
      vars:
        package: httpd
        service: service
```

## 14、只执行指定 tag 并且有 tag 时执行

```
- name: clean
  import_tasks: clean.yml
  when: "'clean' in ansible_run_tags"
  tags:
    - clean
```

## 15、delegate_to

```
- name: 创建 local storage 本地目录
  shell:
    cmd: |
      rm -rf {{ local_path }}
      mkdir -p {{ local_path }}
  delegate_to: "{{ item }}"
  loop: "{{ groups['tmp'] }}"
```

## 16、when

```
- hosts: localhost
  roles:
  - cluster-addon
  - { role: tmp, when: enable_tmp=='yes' }
```

```
when: label is defined and label != ""
```

```
when: (enable_tmp is defined and enable_tmp == 'yes') or (enable_all is defined and enable_all == 'yes')
```

## 17、ignore_unreachable | ignore_errors: true

```
- name: 清理 /etc/hosts
  shell:
    cmd: |
      echo "{{ conf }}"|grep address|cut -d'/' -f2|xargs -i sed -i "/{}/d" /etc/hosts
  delegate_to: "{{ item }}"
  ignore_unreachable: yes
  ignore_errors: true
  with_items:
    - "{{ server }}"
    - "{{ node }}"
```

## 18、copy

```
- name: 分发 resolv.conf
  copy: src={{ dir }}/yml/{{ name }}/resolv.conf dest=/etc/resolv.conf
  delegate_to: "{{ item }}"
  with_items:
    - "{{ server }}"
    - "{{ node }}"

```

## 19、restart

```
- name: 重启服务端 dnsmasq
  service:
    name: dnsmasq
    state: restarted
    use: systemd
    enabled: yes
  delegate_to: "{{ item }}"
  loop: "{{ server }}"
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

