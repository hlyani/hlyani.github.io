# git 相关

# 一、git

## 1.Usual

```
git config --global credential.helper store

git fetch --all && git reset --hard origin/master

# 获取分支名，以 '/' 拆分，取最后一个值
git rev-parse --abbrev-ref HEAD | awk -F'/' '{print $NF}'

git format-patch -s -1

git reset --hard       # 撤销所有已暂存和未暂存的更改（基于最新提交）
git clean -fd          # 删除所有未跟踪的文件和目录（-f 强制，-d 包括目录）
```

## 2.Other

### 1、Git 配置

```
/etc/gitconfig 所有用户，git config --system
~/.gitconfig 当前用户，git config --global
.git/config

#覆盖优先级
.git/config > ~/.gitconfig > /etc/gitconfig 

git config --local http.postBuffer 524288000
git config --global http.lowSpeedLimit 0
git config --global http.lowSpeedTime 999999
git config --global user.name "hl"
git config --global core.editor vim
git config --global merge.tool vimdiff
git config --list

#15 分钟
git config --global credential.helper cache

#永久
git config --global credential.helper store

#自定义时间
git config credential.helper 'cache --timeout=3600'

#删除远程分支
git push origin --delete feature-x
git push origin :feature-x

git fetch --all --prune

#撤销git add 添加的文件
git reset
git checkout -- <file-or-directory>

#撤销git add 添加的文件，同时丢弃add中的更改
git reset --hard
git reset --hard HEAD^

#Git使用了一个内部数据库来存储对象（如提交、树和blob）。随着时间的推移，这个数据库可能会包含一些不再被任何分支或标签引用的对象。
git gc
git gc --prune=now

# 列出所有远程分支
git branch -r

# 列出本地和远程分支
git branch -a

# 查看某段代码是谁写的
git blame  <file-name>

# 跳过 ssl 验证
git config --global http.sslVerify false

# 更新 author
git commit --amend --author="hl <hl@tmp.com>" --no-edit
```
### 2、查看状态
```
git status -s
git log --oneline
git tags
git branch
```
### 3、rebase
```
git fetch --all
git rebase origin/master

git rebase -i HEAD~4

git rebase -i 113sff3e4f
```
### 4、cherry-pick
```
git cherry-pick sdfs32233
git add .
git commit --amend
git cherry-pick --continue
git push origin hl -f
```
### 5、amend
```
git add .
git commit --amend
git rebase continue
git push origin hl -f
```
### 6、checkout
```
git checkout .
git checkout -b dev origin/master  
```
### 7、remote add
```
git remote add test http://192.168.0.1/test/test.git
git remote -vv
```
### 8、format am apply
```
git format-patch -s -2
git am 0001-let-chinese-text-no-wrap.patch

git format-patch 32399a0518ed22bc69ed413310eca4a5f50fa0d1^..8a03a7f472664641023fb08f5d55d16dfb192af9

# 检查是否有冲突
git apply --check 4.2.2.2_to_4.2.2.6/*.patch	
```
```
git format-patch -s -1
git apply --3way 0001-let-chinese-text-no-wrap.patch
# 解决冲突
git add .
git commit -am "xxxx"
```

> --3way 选项会尝试三向合并，若失败则在工作区文件中插入冲突标记。

```
git format-patch -s -1
git am 0001-let-chinese-text-no-wrap.patch
git apply --3way 0001-let-chinese-text-no-wrap.patch
# 解决冲突
git am --continue
```

### 9、reset

```
git reset --hard 7f575c8
```
### 10、reset-author
```
git commit --amend --reset-author 
```
### 11、color
```
git config --global color.status auto  
git config --global color.diff auto  
git config --global color.branch auto  
git config --global color.interactive auto  
```
### 12、log
```
git log --pretty=oneline 32399a0518ed22bc69ed413310eca4a5f50fa0d1..8a03a7f472664641023fb08f5d55d16dfb192af9

git log --pretty=format:"%h; author: %cn; date: %ci; subject:%s" tagA...tagB

比较本地的仓库和远程仓库的区别
git log -p master..origin/master

git log -p
```
### 13、stash
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
### 13、submodule

[submodule](https://segmentfault.com/a/1190000003076028)

```
git submodule add git@github.com:jjz/pod-library.git pod-library
git add .gitmodules pod-ibrary
git commit -m "pod-library submodule"
git submodule init
git push
```

### 14、merge

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

### 15、branch

```
删除temp
git branch -d temp 

查看分支
git branch -vv

git branch --all
```

### 16、diff

```
比较master分支和temp分支的不同
git diff temp
```

### 17、fetch

```
从远程的origin仓库的master分支下载代码到本地的origin master
git fetch origin master

从远程的origin仓库的master分支下载到本地并新建一个分支temp
git fetch origin master:temp
```

### 18、add

```
交互地添加文件至缓存区
git add -i
```

### 19、config

```
彩色的 git 输出：
git config color.ui true

显示历史记录时，只显示一行注释信息：
git config format.pretty oneline
```

### 20、archive

```
# --format 表示打包的格式，如 zip，-v 表示对应的 tag 名，后面跟的是 tag 名，如 v0.1
git archive -v -format=zip v0.1>v0.1.zip

git archive --format=tar --output /full/path/to/zipfile.zip master | gzip
```

### 21、subtree

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

### 22、使用用户名和密码

```
git clone http://邮箱(或用户名):密码@仓库
```

```
git clone -b master --depth 1 http://admin:123@192.168.0.1/demo/demo.git
```

### 23、tag

```
git tag -a "v1.0" -m "v1.0"

推送所有tag
git push origin --tags

删除本地记录
git tag -d v1.0

删除远程记录
git push origin :refs/tags/v1.0
```

### 24、拉取单个分支到指定目录

```
git clone --single-branch --branch=v1.0.0 --depth=1 http://192.168.0.1/test/tmp test
```

### 25、强制推送所有分支

```
需要推送的分支需要每个都 git checkout xx

git push --all origin -f
```

### 26、只拉取单个分支

```
git clone --single-branch --branch=v1.0.0 --depth=1 http://XXX build/src/XXX
```

### 27、重置分支

```
git reset --hard origin/master 
```

### 28、统计分支直接的差异

```
git diff --numstat main dev | awk '{add+=$1; del+=$2} END {print "Added:", add, "Deleted:", del, "Total:", add+del}'
```

```
git ls-files | xargs cat | wc -l
```

### 29、忽略空格变化

```
# 使用 --ignore-space-change 选项忽略空格变化
git apply --ignore-space-change ..\0001-feat.patch

# 使用 --ignore-whitespace 选项忽略所有空白字符差异
git apply --ignore-whitespace ..\0001-feat.patch

# 使用 --whitespace=nowarn 选项忽略空白字符警告
git apply --whitespace=nowarn ..\0001-feat.patch
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
```
stages:
  - build
  - upload
  - release

variables:
  PACKAGE_VERSION: ${CI_COMMIT_BRANCH}-${CI_COMMIT_SHORT_SHA}
  REGISTRY: "192.168.0.1"
  DIR: "dist"
  AMD64_TAR: "aa-amd64.tar.gz"
  ARM64_TAR: "aa-arm64.tar.gz"
  PACKAGE_REGISTRY_URL: "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/generic/Release/${PACKAGE_VERSION}"

build-job:
  stage: build
  image: ${REGISTRY}/cicd/build-base:v1.0.1
  rules:
    - if: $CI_COMMIT_TAG # Do not run this job when a tag is created manually
      when: never
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH # Run this job when commits are pushed or merged to the default branch
  script:
    - echo "running build-job for ${PACKAGE_VERSION}"
    - make
    # - yes|docker system prune -a
  artifacts:
    untracked: false
    when: on_success
    expire_in: 30 days
    paths:
      - ${DIR}/${AMD64_TAR}
      - ${DIR}/${ARM64_TAR}

upload-job:
  stage: upload
  image: ${REGISTRY}/cicd/curl:latest
  rules:
    - if: $CI_COMMIT_TAG # Do not run this job when a tag is created manually
      when: never
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH # Run this job when commits are pushed or merged to the default branch
  needs:
    - job: build-job
      artifacts: true
  script:
    - echo "running upload-job for ${PACKAGE_VERSION}"
    - |
      curl --header "JOB-TOKEN: ${CI_JOB_TOKEN}" --upload-file ${DIR}/${AMD64_TAR} "${PACKAGE_REGISTRY_URL}/${AMD64_TAR}"
      curl --header "JOB-TOKEN: ${CI_JOB_TOKEN}" --upload-file ${DIR}/${ARM64_TAR} "${PACKAGE_REGISTRY_URL}/${ARM64_TAR}"
    - echo ${PACKAGE_REGISTRY_URL}/${AMD64_TAR}
    - echo ${PACKAGE_REGISTRY_URL}/${ARM64_TAR}

release-job:
  stage: release
  image: ${REGISTRY}/cicd/release-cli:latest
  rules:
    - if: $CI_COMMIT_TAG # Do not run this job when a tag is created manually
      when: never
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH # Run this job when commits are pushed or merged to the default branch
  needs:
    - job: upload-job
  script:
    - echo "running release-job for ${PACKAGE_VERSION}"
  release:
    name: ${PACKAGE_VERSION}
    tag_name: ${PACKAGE_VERSION}
    description:  ${PACKAGE_VERSION}
    assets:
      links:
        - name: ${AMD64_TAR}
          url: "${PACKAGE_REGISTRY_URL}/${AMD64_TAR}"
          filepath: "/${AMD64_TAR}"
          link_type: package
        - name: ${ARM64_TAR}
          url: "${PACKAGE_REGISTRY_URL}/${ARM64_TAR}"
          filepath: "/${ARM64_TAR}"
          link_type: package
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

## （五）、规范

```
type：commit 的类型
feat：新特性
fix：修改问题
refactor：代码重构
docs：文档修改
style：代码格式修改，注意不是 css 修改
test：测试用例修改
chore：其他修改，比如构建流程，依赖管理
scope：commit 影响的范围，比如：route，component，utils，build……
subject：commit 的概述
body：commit 具体修改内容，可以分为多行
footer：一些备注，通常是 BREAKING CHANGE 或修复的 bug 的链接
```

# 三、Gitlab备份和恢复

## （一）备份

##### 1、修改配置

```
gitlab_rails['manage_backup_path'] = true
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
gitlab_rails['backup_keep_time'] = 604800 #这个是秒，7天的时间
```

##### 2、重新加载配置，让配置生效

```
gitlab-cli reconfigure
gitlab-cli restart
```

##### 3、备份命令

```
/usr/bin/gitlab-rake gitlab:backup:create
```

> 1674018900_2023_01_18_13.5.4_gitlab_backup.tar
>
> 可通过date命令查看
>
> date -d @1674018900

##### 4、其他需要备份文件

```
/etc/gitlab/gitlab.rb 配置文件须备份 
/var/opt/gitlab/nginx/conf nginx配置文件 
/etc/postfix/main.cfpostfix 邮件配置备份
```

##### 5、定时任务自动备份

> 每周备份一次

```
0 1 * * 1 docker exec gitlab sh -c "/usr/bin/gitlab-rake gitlab:backup:create"
```

## （二）恢复

##### 1、停止数据写入任务

```
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
```

##### 2、恢复数据

```
gitlab-rake gitlab:backup:restore BACKUP=1674018900
```

##### 3、重启服务

```
gitlab-ctl restart
```

## （三）只备份代码

```
cd /var/opt/gitlab/git-data/repositories
```

```
/usr/bin/gitlab-rake gitlab:backup:create
```

# 四、autobak

```
cat > /hl/auto_bak.sh << 'EOF'
#!/bin/bash

# Enable strict mode
set -e

# Log file
LOG_FILE="/var/log/auto_bak.log"

# Log function
log() {
    echo "$(date +"%Y-%m-%d %H:%M:%S") $1" | tee -a "$LOG_FILE"
}

# Set working directory
WORK_DIR="/hl"
cd "$WORK_DIR"

# Check if Git is installed
if ! command -v git &> /dev/null; then
    log "Git is not installed. Please install Git and retry."
    exit 1
fi

# Check for uncommitted changes
if [[ -n $(git status --porcelain) ]]; then
    log "Changes detected. Preparing to commit and push."

    # Add all changes
    git add .

    # Commit changes and push
    git commit -am "Auto backup $(date -u +"%Y%m%d%H%M")"
    if git push; then
        log "Changes have been successfully pushed to the remote repository."
    else
        log "Push failed. Please check your network connection or remote repository settings."
        exit 1
    fi
else
    log "No changes detected. Nothing to commit."
fi
EOF

chmod +x /hl/auto_bak.sh
```

```
cluster_branch=hl
git init
git remote add origin https://root:123@gitlab.xxx.com/hl
git checkout -b ${cluster_branch}
git push --set-upstream origin ${cluster_branch}
```

```
crontab -e
crontab -l
crontab -d
```

# 五、FAQ

> error: RPC failed; curl 56 GnuTLS recv error (-9): Error decoding the received TLS packet.
> error: 4254 bytes of body are still expected
> fetch-pack: unexpected disconnect while reading sideband packet
> fatal: early EOF
> fatal: fetch-pack: invalid index-pack output

```
git config --global http.postBuffer 1048576000
git config --global https.postBuffer 1048576000
```

