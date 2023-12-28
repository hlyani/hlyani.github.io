# GitLab

[https://docs.gitlab.com/ee/](https://docs.gitlab.com/ee/)

# 一、安装

## 1、通过docker安装

```
mkdir -p config logs data
```

```
docker run --detach \
  --hostname 192.168.0.10 \
  --env GITLAB_OMNIBUS_CONFIG="external_url 'http://192.168.0.10:30'; gitlab_rails['lfs_enabled'] = true; gitlab_rails['gitlab_shell_ssh_port'] = 22" \
  --publish 16443:443 --publish 30:30 --publish 1622:22 \
  --name gitlab \
  --restart always \
  --volume ${PWD}/config:/etc/gitlab:Z \
  --volume ${PWD}/logs:/var/log/gitlab:Z \
  --volume ${PWD}/data:/var/opt/gitlab:Z \
  --shm-size 256m \
  gitlab/gitlab-ce:16.6.2-ce.0
```

# 2、获取root密码

```
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

## 3、其他

```
docker exec -it gitlab editor /etc/gitlab/gitlab.rb

docker exec gitlab gitlab-ctl reconfigure

docker exec gitlab update-permissions
```

# 二、配置runner

## 1、通过虚拟机

> 安装 gitlab-runner

```
curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
```

```
gitlab-runner register
              
Enter the GitLab instance URL (for example, https://gitlab.com/):
http://192.168.0.10:30/
Enter the registration token: (从gitlab管理界面获取token)
GR1348951A12mTBjNFzr9FXcH8sR1
Enter a description for the runner:
[runner]: vm
Enter tags for the runner (comma-separated):
vm
Enter optional maintenance note for the runner:
Enter an executor: shell, ssh, docker+machine, instance, kubernetes, custom, docker, docker-windows, parallels, virtualbox, docker-autoscaler:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
Configuration (with the authentication token) was saved in "/etc/gitlab-runner/config.toml" 
```

```
cat /etc/gitlab-runner/config.toml

concurrent = 1
check_interval = 0
shutdown_timeout = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "vm22"
  url = "http://192.168.0.10:30/"
  id = 4
  token = "asdsad"
  token_obtained_at = 2023-12-15T03:23:09Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "ssh"
  [runners.cache]
    MaxUploadedArchiveSize = 0
  [runners.ssh]
    user = "root"
    password = "123"
    host = "192.168.0.20"
    port = "22"
```

## 2、runner 依赖安装

> 安装 release-cli

```
curl --location --output /usr/local/bin/release-cli "https://gitlab.com/api/v4/projects/gitlab-org%2Frelease-cli/packages/generic/release-cli/latest/release-cli-linux-amd64"

chmod +x /usr/local/bin/release-cli
```

> 安装 docker 环境
>
> 安装 golang 环境
>
> 安装 buildx 环境

## 3、其他配置

> gitlab 管理界面   CI/CD -> Runners -> 编辑（铅笔图标） -> Run untagged jobs （勾选）

# 三、编写 .gitlab-ci.yml

```
stages:
  - build
  - upload
  - release

variables:
  PACKAGE_VERSION: ${CI_COMMIT_BRANCH}-${CI_COMMIT_SHORT_SHA}
  REGISTRY: "192.168.0.10:3000"
  AMD64_TAR: "aaa-amd64.tar.gz"
  ARM64_TAR: "aaa-arm64.tar.gz"
  INSTALL: "install.sh"
  PACKAGE_REGISTRY_URL: "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/generic/Release/${PACKAGE_VERSION}"

build-job:
  image: ${REGISTRY}/cicd/build-base:v1.0.1
  stage: build
  except: 
    - tags
  # rules:
  #   - if: $CI_COMMIT_TAG # Do not run this job when a tag is created manually
  #     when: never
  #   - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH # Run this job when commits are pushed or merged to the default branch
  script:
    - echo "running build-job for ${PACKAGE_VERSION}"
    - make
  artifacts:
    untracked: false
    when: on_success
    expire_in: 30 days
    paths:
      - bin/${AMD64_TAR}
      - bin/${ARM64_TAR}
      - bin/${INSTALL}

upload-job:
  stage: upload
  image: ${REGISTRY}/cicd/curl:latest
  needs:
    - job: build-job
      artifacts: true
  script:
    - echo "running upload-job for ${PACKAGE_VERSION}"
    - |
      curl --header "JOB-TOKEN: ${CI_JOB_TOKEN}" --upload-file bin/${AMD64_TAR} "${PACKAGE_REGISTRY_URL}/${AMD64_TAR}"
      curl --header "JOB-TOKEN: ${CI_JOB_TOKEN}" --upload-file bin/${ARM64_TAR} "${PACKAGE_REGISTRY_URL}/${ARM64_TAR}"
      curl --header "JOB-TOKEN: ${CI_JOB_TOKEN}" --upload-file bin/install.sh "${PACKAGE_REGISTRY_URL}/${INSTALL}"
    - echo ${PACKAGE_REGISTRY_URL}/${AMD64_TAR}
    - echo ${PACKAGE_REGISTRY_URL}/${ARM64_TAR}
    - echo ${PACKAGE_REGISTRY_URL}/${INSTALL}

release-job:
  stage: release
  image: ${REGISTRY}/cicd/release-cli:latest
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
        - name: ${INSTALL}
          url: "${PACKAGE_REGISTRY_URL}/${INSTALL}"
          filepath: "/${INSTALL}"
          link_type: package
```

