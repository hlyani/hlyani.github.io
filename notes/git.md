# git 相关

# 一、git

##### 1、Git 配置
```
/etc/gitconfig 文件：系统中对所有用户都普遍适用的配置。若使用 git config 时用 --system 选项，读写的就是这个文件。

~/.gitconfig 文件：用户目录下的配置文件只适用于该用户。若使用 git config 时用 --global 选项，读写的就是这个文件。

.git/config

当前项目的 Git 目录中的配置文件（也就是工作目录中的 .git/config 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 .git/config 里的配置会覆盖 /etc/gitconfig 中的同名变量。

git config --local http.postBuffer 524288000
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999

git config --global user.name "hl"
git config --global core.editor vim
git config --global merge.tool vimdiff
git config --list

#设置记住密码（默认15分钟）：
git config --global credential.helper cache

#永久
git config --global credential.helper store

#如果想自己设置时间，可以这样做：
git config credential.helper 'cache --timeout=3600'
```
##### 2、查看状态
```
git status -s
```
##### 3、rebase
```
git fetch --all
git rebase origin/master

git rebase -i HEAD~4

git rebase -i 113sff3e4f
```
##### 4、cherry-pick
```
git cherry-pick sdfs32233
git add .
git commit --amend
git  cherry-pick --continue
git push origin hl -f
```
##### 5、amend
```
git add .
git commit --amend
git rebase continue
git push origin hl -f
```
##### 6、checkout
```
git checkout .

git checkout -b dev origin/master  
```
##### 7、remote add
```
git remote add test http://192.168.21.1/test/test.git

git remote -vv
```
##### 8、format am apply
```
git format-patch -s -2
git am 0001-let-chinese-text-no-wrap.patch

git format-patch 32399a0518ed22bc69ed413310eca4a5f50fa0d1^..8a03a7f472664641023fb08f5d55d16dfb192af9

git apply --check 4.2.2.2_to_4.2.2.6/*.patch	
```
##### 9、reset
```
git reset --hard 7f575c8
```
##### 10、reset-author
```
git commit --amend --reset-author 
```
##### 11、color
```
git config --global color.status auto  
git config --global color.diff auto  
git config --global color.branch auto  
git config --global color.interactive auto  
```
##### 12、log
```
git log --pretty=oneline 32399a0518ed22bc69ed413310eca4a5f50fa0d1..8a03a7f472664641023fb08f5d55d16dfb192af9

git log --pretty=format:"%h; author: %cn; date: %ci; subject:%s" tagA...tagB

比较本地的仓库和远程仓库的区别
git log -p master..origin/master

git log -p
```
##### 13、stash
```
git stash
git stash save "test-cmd-stash"
git stash list
git stash drop
# 删除stash暂存区
git stash pop
# 不删除stash暂存区，可多次应用
git stash apply
# 清空栈中所有记录
git stash clear
```
##### 13、submodule

[submodule](https://segmentfault.com/a/1190000003076028)

```
git submodule add git@github.com:jjz/pod-library.git pod-library
git add .gitmodules pod-ibrary
git commit -m "pod-library submodule"
git submodule init
git push
```

##### 14、merge

```
合并dev分支到master分支

git checkout dev
git pull
git checkout master
git merge dev
git push -u origin master

把远程下载下来的代码合并到本地仓库，远程的和本地的合并
git merge origin/master
```

##### 15、branch

```
删除temp
git branch -d temp 

查看分支
git branch -vv

git branch --all
```

##### 16、diff

```
比较master分支和temp分支的不同
git diff temp
```

##### 17、fetch

```
从远程的origin仓库的master分支下载代码到本地的origin master
git fetch origin master

从远程的origin仓库的master分支下载到本地并新建一个分支temp
git fetch origin master:temp
```

##### 18、add

```
交互地添加文件至缓存区
git add -i
```

##### 19、config

```
彩色的 git 输出：
git config color.ui true

显示历史记录时，只显示一行注释信息：
git config format.pretty oneline
```

##### 20、archive

```
# --format 表示打包的格式，如 zip，-v 表示对应的 tag 名，后面跟的是 tag 名，如 v0.1
git archive -v -format=zip v0.1>v0.1.zip
```

##### 21、subtree

```
git subtree pull --prefix=<子目录名> <远程分支> <分支> 

#将 B 仓库添加为 A 仓库的一个子目录
git subtree add --prefix=SubFolder/B https://github.com/walterlv/walterlv.git master

#将 A 仓库中的 B 子目录推送回 B 仓库
git subtree push --prefix=SubFolder/B https://github.com/walterlv/walterlv.git master

#将 B 仓库中的新内容拉回 A 仓库的子目录
git subtree pull --prefix=SubFolder/B walterlv master

# 将需要分离的目录的提交日志分离成一个独立的临时版本
git subtree split -P <name-of-folder> -b <name-of-new-branch>

# example

- name: Publish
uses: hlyani/gitbook-deploy-action@master
env:
    ACCESS_TOKEN: ${{ secrets.testaction }}
    BASE_BRANCH: source
    BRANCH: master
    FOLDER: docs

git add -f $FOLDER

git commit -m "Deploying to ${BRANCH} from ${BASE_BRANCH:-master} ${GITHUB_SHA}" --quiet

git push $REPOSITORY_PATH `git subtree split --prefix $FOLDER ${BASE_BRANCH:-master}`:$BRANCH --force 
```

##### 22、使用用户名和密码

```
git clone http://邮箱(或用户名):密码@仓库
```

```
git clone -b master --depth 1 http://admin:123456@192.168.0.1/demo/demo.git
```

# 二、常用

## （一）、修改之前的commit

##### 1、将HEAD移动到需要修改的commit上

```
git rebase eb69dddd^ --interactive
```

##### 2、找到需要修改的commit,将首行的pick改成edit后保存

##### 3、开始修改内容

##### 4、添加改动文件到暂存

```
git add
```

##### 5、追加改动到提交

```
git commit --amend
```

##### 6、移动HEAD回到最新的commit

```
git rebase --continue
```

##### 7、强制提交

```
git push origin master -f
```

## （二）、合并 commit

##### 1、找到要合并 commit 的前一个commit

```
git rebase -i sdfs1easdasd

# git rebase -i HEAD~3
```

##### 2、将要合并的 commit 前改为 squash

```
pick asdasd Add second commit
squash asdasd Add third commit

:wq
```

##### 3、操作失误 --abort 撤销

```
git rebase --abort
```

##### 4、提交

```
git add .
git push origin master -f
```

## （三）、lfs

##### 1、安装

```
yum install git-lfs

apt install git-lfs

git lfs install
```

##### 2、标记大文件

```
git lfs track kernel_resource/4.19.37-rt19.arm64.gz

git lfs track *.gz
```

##### 3、推送

```
 git add my.gz
 git commit -m "add gz file"
 git push origin master
```

##### 4、查看

```
git lfs track
git lfs ls-files
vim .gitattributes
```

## （四）、gitlab-ci

##### 1、拉取gitlab容器镜像
```
docker pull gitlab/gitlab-runner:latest
```
##### 2、运行gitlab
```
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest

mkdir -p /home/hl/gitlabrunner/DeltaOS/opt /home/hl/gitlabrunner/DeltaOS/builds

docker run -d --name deltaos-gitlab-runner --restart always \
  --privileged=true \
  -v /home/hl/gitlabrunner/DeltaOS/opt:/opt \
  -v /home/hl/gitlabrunner/DeltaOS/builds:/home/gitlab-runner/builds \
  gitlab/gitlab-runner:latest

docker exec deltaos-gitlab-runner gitlab-runner register \
 --non-interactive \
 --name my-runner \
 --url http://192.168.1.1:8000/ \
 --registration-token sDrr7KDi6UXyzwuzPqx- \
 --executor shell \
 --tag-list \
 common-runner
```
##### 3、容器内
```
mknod -m 0660 /dev/loop1 b 7 1
echo "192.168.21.8 gitlab.tmp.com" |tee >> /etc/hosts
#添加root权限
sed -i 's/gitlab-runner:x:999:999/gitlab-runner:x:0:0/g' /etc/passwd
```
##### 4、编辑gitlab-ci
```
vim .gitlab-ci.yml

stages:
  - deploy
deploy:
    stage: deploy
    script:
      - bash /opt/deploy.sh
    only:
      - master
    tags:
      - common-runner

when 可以设置为以下值之一：

on_success - 只有当前一个阶段的所有工作成功时才​​执行工作。这是默认值。
on_failure - 仅当前一个阶段的至少一个作业发生故障时才执行作业。
always - 无论前一阶段的工作状况如何，执行工作。
manual - 手动执行作业

stages:
 - build
 - test
 - deploy

首先，所有build的jobs都是并行执行的。
所有build的jobs执行成功后，test的jobs才会开始并行执行。
所有test的jobs执行成功，deploy的jobs才会开始并行执行。
所有的deploy的jobs执行成功，commit才会标记为success
任何一个前置的jobs失败了，commit会标记为failed并且下一个stages的jobs都不会执行。

有时，script命令需要用单引号或双引号括起来。例如，包含冒号（:）的命令需要用引号括起来，以便YAML解析器知道将整个事物解释为字符串而不是“key：value”对。使用特殊字符时要小心：:，{，}，[，]，,，&，*，#，?，|，-，<，>，=，!，%，@，`。

例如：
stages:
- build
- cleanup_build
- test
- deploy
- cleanup

build_job:
  stage: build
  script:
  - make build

cleanup_build_job:
  stage: cleanup_build
  script:
  - cleanup build when failed
  when: on_failure

test_job:
  stage: test
  script:
  - make test

deploy_job:
  stage: deploy
  script:
  - make deploy
  when: manual

cleanup_job:
  stage: cleanup
  script:
  - cleanup after jobs
  when: always
以上脚本将：

cleanup_build_job仅在build_job失败时执行。
始终执行cleanup_job作为流水线的最后一步，无论成功或失败。
允许您deploy_job从GitLab的UI 手动执行。
```
##### 5、界面查找url和token
```
Settings —> Pipelines —> Specific Runners
```
##### 6、注册
```
gitlab-runner register --non-interactive --name my-runner --url http://192.168.21.8:8000/ --registration-token Kgu3z28pqfZEZsBmquUh --executor shell --tag-list common-runner

vim /etc/passwd
gitlab-runner:x:0:0:GitLab Runner:/home/gitlab-runner:/bin/bash

#手动注册
docker exec -it gitlab-runner gitlab-ci-multi-runner register

#引导会让你输入gitlab的url，输入自己的url，例如http://gitlab.example.com/

#引导会让你输入token，去相应的项目下找到token，例如ase12c235qazd32

#引导会让你输入tag，一个项目可能有多个runner，是根据tag来区别runner的，输入若干个就好了，比如web,hook,deploy

#引导会让你输入executor，这个是要用什么方式来执行脚本，图方便输入shell就好了。
```
##### 7、查看gitlab-runner
```
gitlab-runner list
```
##### 8、宿主机安装
```
yum 安装
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | bash
yum install -y gitlab-ci-multi-runner

echo '[gitlab-ci-multi-runner]
name=gitlab-ci-multi-runner
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ci-multi-runner/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key' |tee >> /etc/yum.repos.d/gitlab-ci-multi-runner.repo

yum makecache
yum install -y gitlab-ci-multi-runner

#设置gitlab-runner root权限
gitlab-runner:x:0:0:GitLab Runner:/home/gitlab-runner:/bin/bash
```
##### 9、其他
```
jenkins
http://blog.csdn.net/abcdocker/article/details/53840629

https://docs.gitlab.com/ee/ci/yaml/README.html
```
