---
title: Golang-Gitlab-CI
date: 2018-11-19 16:08:24
categories:
- CI/CD
tags:
- Golang
- Gitlab
- CI
comments: true
---

## 基本概念

* Pipeline
    构建任务，可以包含多个流程(Stage)，比如安装依赖、编译、测试、部署等
    任何提交和Merge操作都会触发Pipeline

* Stage
    构建阶段，即上述的流程
    所有的Stage**按顺序执行**，某一个Stage执行失败后，不再执行后续的Stage
    所有Stage执行成功之后，Pipeline才算成功

* Job
    Stage中的具体任务
    所有的Job**并行执行**，某一个Job执行失败，则Stage执行失败
    所有的Job执行成功之后，Stage才算成功

* Runner
    Runner是构建任务的实际执行者，通过项目根目录下的`.gitlab-ci.yml`文件来告知Runner要执行的构建任务

  * 根关键字
    * image: 声明构建任务的runner使用的docker镜像地址
    * services: docker镜像使用的服务，通过链接的方式来调用所需要的服务
    * stages: 定义构建任务包含哪些阶段，其中元素的顺序决定了对应各个Stage执行的顺序
    * before_script: 每一个Job执行之前的脚本
    * after_script: 每一个Job执行之后的脚本
    * variables: 定义变量
    * cache: 定义与后续Job之间应缓存的文件，全局缓存，所有的Job都可以使用该缓存

  * Job中的关键字
    * image: 任务使用的docker镜像，为空时使用根中的定义
    * services: 任务中docker镜像使用的服务
    * stage: 任务所属的阶段
    * before_script: Job运行之前的脚本
    * script: Job执行的脚本，必须填写
    * after_script: Job运行之后的脚本
    * variables: 任务中使用的变量
    * cache: 与后续Job之间应缓存的文件
    * only: 指定应用的git分支，可以是分支名称，也可以是tags来指定增加标签的分支
    * except: 排除应用的git分支
    * tags: 指定执行Gitlab-Runner的标签
    * allow_failure: 是否允许失败，默认不允许失败
    * when: 定义何时执行任务 on_success | on_failure | always | manual
    * dependencies: 定义Job依赖关系，这样他们就可以互相传递artifacts
    * artifacts: 定义哪些目录会被压缩和打包
    * environment: 定义环境
    * coverage: 定义给定Job的代码覆盖率设置

  * 缓存配置
    * untracked: 缓存未被git跟踪的文件
    * paths: 缓存目录和文件
    * policy: 缓存策略 push push-pull pull
    * key: 设置缓存的Key，在有多个缓存的时候需要设置，否则缓存内容会被重写

* Glide-Golang包管理

  * 安装: go get -u -v github.com/Masterminds/glide
  
  * 初始化: glide init
      glide会对代码中依赖的golang包做静态扫描，并生成`glide.yaml`文件

  * 下载包: glide install
      glide会将依赖的包下载到vendor目录中

  * 镜像和下载subpackage: 如果需要下载的包国内无法访问或者需要下载某个subpackage，可以在`glide.yaml`中添加如下配置

    ```shell
    - package: golang.org/x/image
    repo: https://github.com/golang/image //指明镜像仓库地址
    vcs: git
    subpackages: //指明要下载的子包
    - bmp
    - draw
    - font
    - math/f64
    - math/fixed
    ```

  * 私有仓库配置: 如果代码依赖私有仓库中的项目，则可以通过git submodules功能来配置依赖
    * `git submodule add git@somewhere:folder/mysubmodule.git`: 添加submodule
    * 修改.gitmodules文件，将url改为相对路径

    ```shell
    [submodule "mysubmodule"]
    path = mysubmodule
    url = ../../group/mysubmodule.git
    ```

    * 在gitlab-ci.yml中配置获取submodule

    ```shell
    variables:
    GIT_SUBMODULE_STRATEGY: recursive
    ```
