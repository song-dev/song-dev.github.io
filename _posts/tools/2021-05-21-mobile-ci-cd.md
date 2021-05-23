---
layout: post
title: "移动端CI&CD方案探索"
subtitle: 'Jenkins & fastlane 部署 CI\CD 流程'
author: "Song"
header-style: text
tags:
  - markdown
  - pdf
  - pandoc
  - mdout
---

## 概念

- 持续集成指的是，频繁地（一天多次）将代码集成到主干。持续集成的目的，就是让产品可以快速迭代，同时还能保持高质量。它的核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。
- 持续交付（Continuous delivery）指的是，频繁地将软件的新版本，交付给质量团队或者用户，以供评审。如果评审通过，代码就进入生产阶段。持续交付可以看作持续集成的下一步。它强调的是，不管怎么更新，软件是随时随地可以交付的。
- 持续部署（continuous deployment）是持续交付的下一步，指的是代码通过评审以后，自动部署到生产环境。

## 方案探索

1. 使用[Jenkins](https://jenkins.io/zh/) + [fastlane](https://docs.fastlane.tools/)
2. 使用[GitLab](https://docs.gitlab.com/ee/ci/) + [fastlane](https://docs.fastlane.tools/)

Jenkins是开源CI&CD软件领导者，提供超过1000个插件来支持构建、部署、自动化，满足任何项目的需要 

- 作为一个可扩展的自动化服务器，Jenkins可以用作简单的CI服务器，或者变成任何项目的连续交付中心；
- Jenkins是一个独立的基于Java的程序，可以立即运行，包含Windows，Mac OS X和其他类Unix操作系统；
- Jenkins可以通过其网页界面轻松设置和配置，其中包括即时错误检查和内置帮助；
- 通过更新中心中的1000多个插件，Jenkins集成了持续集成和持续交付工具链中几乎所有的工具；
- Jenkins 可以通过其插件架构进行扩展，从而为 Jenkins 可以做的事提供几乎无限的可能性；
- Jenkins可以轻松地在多台机器上分配工作，帮助更快速地跨多个平台推动构建，测试和部署

GitLab CI/CD是GitLab自带的CI&CD工具

## 方案分析

1. Jenkins服务控制用户管理、源码管理、构建时机、构建执行以及构建后的上传、归档、邮件通知等操作，fastlane负责具体的构建操作。Jenkins服务启动之后，可以新建一个任务，通过任务执行构建操作。点击构建之后，Jenkins会创建一个workspace，然后根据Git配置，将源码从Git拉到对应任务的workspace中，代码拉完之后，通过fastlane脚本即可执行构建，构建完成之后，Jenkins可以将构建生成的安装包或者SDK进行归档，然后邮件通知到相关开发人员和测试人员；
2. GitLab自带CI&CD工具，在项目根目录创建一个.gitlab-ci.yml文件，GitLab就会自动将项目纳入CI&CD控制。GitLab要执行构建，需要宿主机(需要执行构建的机器)先起一个Runner的服务，并注册到GitLab，GitLab通过Runner调用.gitlab-ci.yml中的构建脚本执行构建，具体构建的执行，也是通过fastlane脚本实现；
3. fastlane依赖于Xcode command line tools和gradle来执行iOS和Android的构建，所以iOS的CI&CD必须依赖于Mac OS

## 方案选择

通过GitLab实现CI&CD非常方便简洁，只需在宿主机起一个Runner服务并注册到GitLab，但是由于公司现在没有连接外网的Mac机器，而公司的GitLab又是部署在外网的，GitLab无法直接与内网的Mac机器通信，就必须将Mac的IP映射到外网，然后在GitLab中设置如何通过映射来与Mac机器通信，从运维的角度，实现难度较高，另一方面，通过GitLab方式实现CI&CD的构建后的归档、邮件通知等操作，也不甚熟悉，故最终选择通过[Jenkins](https://jenkins.io/zh/) + [fastlane](https://docs.fastlane.tools/)来实现CI&CD。

## 方案实现

### 环境准备

1. 安装Xcode、Android Studio、Xcode command line tools、gradle
2. 安装Jenkins
3. 安装fastlane
4. 其他依赖环境安装

具体安装教程可Google

### 脚本编写

编写脚本之前，需要先熟悉fastlane有哪些功能，具体可以通过[fastlane](https://docs.fastlane.tools/)官网查看，也可以在安装完fastlane之后，通过命令`fastlane actions`来查看，如下图：

![fastlane_actions.png](http://note.youdao.com/yws/res/12905/WEBRESOURCE9b3d74e97459730adfd1fbe8506b6c4b)

1. iOS的Fastfile编写

    打开终端，cd到工程根目录，执行`fastlane init`，fastlane会在根目录新建一个fastlane目录，fastlane目录中会有一个Fastfile文件，通过vim打开此文件，或者通过Sublime Text等工具打开此文件，根据具体工程情况进行修改，如，OneLoginDemo的Fastfile如下：
    
    ```
    default_platform(:ios)
    
    platform :ios do
      desc "Description of what the lane does"
      lane :build do
      	workspace = "OneLoginDemo.xcworkspace"
      	xcodeproj = "OneLoginDemo.xcodeproj"
      	scheme = "OneLoginDemo"
      	app_id = "com.geetest.onelogin"
    
      	version = get_version_number(xcodeproj: xcodeproj)
      	echo(message: "------------- version ------------")
      	echo(message: version) 
      	echo(message: "------------- version ------------")
    
      	build_number = Time.now.strftime("%y%m%d%H%M")
      	echo(message: "------------- build_number ------------")
      	echo(message: build_number)
      	echo(message: "------------- build_number ------------")
    
      	ipa_name = "iOS-OneLogin-" + version + "-" + build_number + ".ipa"
      	echo(message: "------------- ipa_name ------------")
      	echo(message: ipa_name)
      	echo(message: "------------- ipa_name ------------")
    
      	update_app_identifier(
          xcodeproj: xcodeproj,
          plist_path: "OneLoginDemo/Info.plist",
          app_identifier: app_id
        )
    
        increment_build_number(
          xcodeproj: xcodeproj,
          build_number: build_number
        )
    
        cocoapods(podfile: "Podfile")
    
        build_app(scheme: scheme, 
        	      workspace: workspace,
        	      include_bitcode: false, 
        	      clean: true, 
        	      output_directory: "out", 
        	      output_name: ipa_name,
        	      export_method: "development")
    
        # 上传到蒲公英
        # pgyer(api_key: "f2afcec3bc3d07af303921171ca0d659", user_key: "ff5e2bcd4aa013501dea5c2201de8a1a")
    
      end
    end
    ```
    
2. iOS编译脚本
    在工程根目录，新建build.sh文件，编写执行fastlane的脚本，如下：
    
    ```shell
    # 更新fastlane
    bundle update fastlane

    # 删除原来的ipa
    out_dir="out"
    if [[ -d "$out_dir" ]]; then
	   echo "----------- delete out folder ------------"
	   rm -r -f $out_dir
	   echo "----------- delete out folder ------------"
    fi

    # 编译新的ipa
    fastlane build
    ```
    
3. 测试脚本，打开终端，cd到工程根目录，执行`./build.sh`，看执行结果是否正常
4. Android的Fastfile编写
  
    打开终端，cd到工程根目录，执行`fastlane init`，fastlane会在根目录新建一个fastlane目录，fastlane目录中会有一个Fastfile文件，通过vim打开此文件，或者通过Sublime Text等工具打开此文件，根据具体工程情况进行修改，如，OneLoginDemo的Fastfile如下：
    
    ```
    default_platform(:android)

    platform :android do
        desc "Generate apk and aar"
        lane :build do
        gradle(
            task: "assemble",
            build_type: "Release"
        )
        end
    end
    ```
5. Android编译脚本
    在工程根目录，新建build.sh文件，编写执行fastlane的脚本，如下：
    
    ```shell
    # 更新fastlane
    bundle update fastlane

    # 删除原有的aar
    rm -f -r onelogin/build/outputs/aar/*.aar

    # 编译新的apk
    fastlane build

    # 重命名aar
    cd onelogin

    version=`gradle -q printVersionName`
    echo -------- version ---------
    echo ${version}
    echo -------- version ---------

    time=$(date "+%y%m%d%H%M")
    echo -------- time ---------
    echo ${time}
    echo -------- time ---------

    cd build/outputs/aar

    aarname="geetest_onelogin_android_v"${version}_${time}
    echo -------- aarname ---------
    echo ${aarname}
    echo -------- aarname ---------

    # aar重命名
    mv -f -v onelogin.aar ${aarname}.aar
    ```
    
6. 测试脚本，打开终端，cd到工程根目录，执行`./build.sh`，看执行结果是否正常

### Jenkins配置

1. 启动Jenkins服务，通过`nohup java -jar jenkins.war --httpPort=8081`即可启动Jenkins服务，注意：jenkins.war要为绝对路径，可将此行命令放到build.sh文件中，后续通过./build.sh即可启动Jenkins服务
2. 启动之后，打开浏览器，输入http://localhost:8081即可访问Jenkins服务，初次访问，需要登录，密码根据提示去对应的目录下获取即可
3. 插件安装，以下为一些必要插件，有些插件Jenkins已默认安装，可不必重复安装

    Email Extension Plugin</br>
    Environment Injector Plugin</br>
    GitLab Plugin</br>
    Gradle Plugin</br>
    Localization: Chinese(Simplified)</br>
    PostBuildScript Plugin</br>
    PowerShell plugin</br>
    Upload to pgyer</br>
4. 新建任务，新建任务，对任务进行配置，如下图：
  
    ![jenkins_config_01.png](http://note.youdao.com/yws/res/12909/WEBRESOURCEc1221330c3c806613109f7fafba4f39c)    
    
    ![jenkins_config_02.png](http://note.youdao.com/yws/res/12913/WEBRESOURCE4914626c98a286e320436f30cd908a12)
    
    ![jenkins_config_03.png](http://note.youdao.com/yws/res/12916/WEBRESOURCE615b6205621a004ab762f69623385bff)
    
    ![jenkins_config_04.png](http://note.youdao.com/yws/res/12919/WEBRESOURCEa5b4e9de9b90399790075f0a0e3132ee)
    
5. 配置完成后，即可执行构建

    ![jenkins_build.png](http://note.youdao.com/yws/res/12922/WEBRESOURCE9f63eb26002c9a29df0c3a7c7218a690)
    
### Jenkins配置注意事项

1. 要在构建完成之后，通过邮件通知相关人员，一定要先安装Email Extension Plugin插件，然后到系统设置中，配置发送邮件用的服务邮箱，并且测试邮件是否能发送成功。

    ![jenkins_email_config_01.png](http://note.youdao.com/yws/res/12925/WEBRESOURCE1a3d1a5c58bfb1865529c89c0782012d)
    
    ![jenkins_email_config_02.png](http://note.youdao.com/yws/res/12928/WEBRESOURCEaea97812ab6ae0f1daf33fe8e7b15c9b)
    
    ![jenkins_email_config_03.png](http://note.youdao.com/yws/res/12931/WEBRESOURCE37c5d67b8445b14cb515ddb293ae6ba2)
    
2. 若想在构建完之后显示对应的二维码，可在构建后操作中增加Set build description选项，然后设置Description为img标签，通过H5显示二维码，但是因为Jenkins出于安全的考虑，所有描述信息的Markup Formatter默认都是采用Plain text模式，在这种模式下是不会对build描述信息中的HTML编码进行解析的，所以需要到全局安全配置中将Markup Formatter改为Safe HTML。
  
    ![jenkins_markupformatter_config_01.png](http://note.youdao.com/yws/res/12935/WEBRESOURCEc22db8e0d95372e5a3a182a9c584ba6f)

    ![jenkins_markupformatter_config_02.png](http://note.youdao.com/yws/res/12938/WEBRESOURCE421fbaa66dda2850fee3d39355c84524)
    

**附Jenkins配置参考链接：**

- [https://www.jianshu.com/p/db5fe7fed9f3](https://www.jianshu.com/p/db5fe7fed9f3)
- [https://www.jianshu.com/p/3511625b2ffc](https://www.jianshu.com/p/3511625b2ffc)
- [https://github.com/whihail/AutoArchive/wiki/%E5%AE%A2%E6%88%B7%E7%AB%AFJenkins%E8%87%AA%E5%8A%A8%E6%9E%84%E5%BB%BA%E6%8C%87%E5%8D%97%E4%B9%8B%E9%82%AE%E4%BB%B6%E9%80%9A%E7%9F%A5](https://github.com/whihail/AutoArchive/wiki/%E5%AE%A2%E6%88%B7%E7%AB%AFJenkins%E8%87%AA%E5%8A%A8%E6%9E%84%E5%BB%BA%E6%8C%87%E5%8D%97%E4%B9%8B%E9%82%AE%E4%BB%B6%E9%80%9A%E7%9F%A5)
- [https://www.jianshu.com/p/73456eca01ae](https://www.jianshu.com/p/73456eca01ae)
- [https://blog.csdn.net/hanjiangong666/article/details/80963591](https://blog.csdn.net/hanjiangong666/article/details/80963591)

## 容器化

### 为什么要容器化

软件开发最大的麻烦事之一，就是环境配置。用户计算机的环境都不相同，你怎么知道自家的软件，能在那些机器跑起来？

用户必须保证两件事：操作系统的设置，各种库和组件的安装。只有它们都正确，软件才能运行。举例来说，安装一个 Python 应用，计算机必须有 Python 引擎，还必须有各种依赖，可能还要配置环境变量。

如果某些老旧的模块与当前环境不兼容，那就麻烦了。开发者常常会说："它在我的机器可以跑了"（It works on my machine），言下之意就是，其他机器很可能跑不了。

环境配置如此麻烦，换一台机器，就要重来一次，旷日费时。很多人想到，能不能从根本上解决问题，软件可以带环境安装？也就是说，安装的时候，把原始环境一模一样地复制过来。

### 虚拟机

虚拟机（virtual machine）就是带环境安装的一种解决方案。它可以在一种操作系统里面运行另一种操作系统，比如在 Windows 系统里面运行 Linux 系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。

虽然用户可以通过虚拟机还原软件的原始环境。但是，这个方案有几个缺点。

- 资源占用多
- 冗余步骤多
- 启动慢

### Linux容器

由于虚拟机存在这些缺点，Linux 发展出了另一种虚拟化技术：Linux 容器（Linux Containers，缩写为 LXC）。

Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。或者说，在正常进程的外面套了一个保护层。对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。

由于容器是进程级别的，相比虚拟机有很多优势。

- 启动快
- 资源占用少
- 体积小

### Docker

Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。

Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

附Docker学习链接：

- [https://yeasy.gitbooks.io/docker_practice/introduction/what.html](https://yeasy.gitbooks.io/docker_practice/introduction/what.html)
- [http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

### 我们为什么不做容器化？

主要原因是由于iOS的构建依赖于Xcode，而Docker上又没有相关的image，借用stackoverflow上的一个回复，如下：

   ![why_not_docker.png](http://note.youdao.com/yws/res/12941/WEBRESOURCE4eaa53a9a37c3807380c036adf511283)