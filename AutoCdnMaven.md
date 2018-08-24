# 基于Gulp实现静态资源自动发布到CDN和Maven仓库的工程实践

:construction_worker:

## 一.本文目标：

* 1.初步认识gulp
* 2.了解maven，实现自动发布maven war包
* 3.了解OSS，实现自动发布cdn资源
* 4.了解前端工程构建

微服务架构下，为了解决前后端分离，前端资源部署的问题，我们采用了maven项目依赖前端war的方式部署web服务，编写插件，利用gulp完成前端项目的打包发布，包括发布maven和cdn资源，实现自动化推送maven war，减少了前端发布流程，自动化推送cdn，省去了手动上传文件的繁琐步骤。

前端同学可具体关注实现细节，了解maven，oss等基本概念，后端同学可以了解前端自动化实现工具gulp，以及node，源码编译打包过程等。

## 二.初步认识gulp

 gulp是前端开发过程中对代码进行构建的工具，是自动化项目的构建利器；她不仅能对网站资源进行优化，而且在开发过程中很多重复的任务能够使用正确的工具自动完成；使用了以后呢，我们不仅可以很愉快的编写代码，还能大大提高我们的工作效率。
 特点：**易用，易学，快速高效**

### 2.1安装

>依赖node运行环境，node安装使用[传送门](https://nodejs.org/en/docs/guides/)
>
>全局安装gulp,在终端执行
>
>`npm install --global gulp`
>
>作为项目的开发依赖（devDependencies）安装：
>
>`npm install --save-dev gulp`

### 2.2常用的gulp api及插件

#### gulp api总共5个

* Gulp.task(name, fn) 用来定义任务的
* Gulp.run(tasks…) 从3.5开始弃用，将在4.0中删除。https://github.com/gulpjs/gulp/blob/master/index.js#L16
* Gulp.src(glob) 用来读取文件
* Gulp.dest(folder) 用来写入文件
* Gulp.watch(glob, fn) 用来监听文件是否改动过

#### gulp常用插件

1. gulp-less: 把less文件转成css文件
2. gulp-clean-css：css文件压缩
3. gulp-uglify：js压缩
4. gulp-concat：js合并
5. gulp-rename：重命名，给js压缩文件添加.min后缀
6. gulp-jshint：js语法检查

### 2.3编译less文件示例

需要安装gulp-less插件，作为项目的开发依赖（devDependencies）安装：
>
>`npm install --save-dev gulp-less`

在gulpfile.js中添加如下任务

```

var less = require('gulp-less');
var path = require('path');
 
gulp.task('less', function () {
  return gulp.src('./less/**/*.less')
    .pipe(less({
      paths: [ path.join(__dirname, 'less', 'includes') ]
    }))
    .pipe(gulp.dest('./public/css'));
});

```

在终端执行
>
>`gulp less`
>
在public/css中可以看到编译后的css文件内容
>
有关gulp的进阶用法，各种插件请访问[GULP中文网](https://www.gulpjs.com.cn/)

## 三.了解maven，编写gulp 任务，实现自动上传war包

### 3.1 Maven

Maven是一个项目管理和综合工具。Maven提供了开发人员构建一个完整的生命周期框架。开发团队可以自动完成项目的基础工具建设，Maven使用标准的目录结构和默认构建生命周期。Apache Maven 是一种创新的软件项目管理工具，提供了一个项目对象模型（POM）文件的新概念来管理项目的构建，相关性和文档。最强大的功能就是能够自动下载项目依赖库。

### 3.2Maven安装使用

#### 3.2.1Maven安装配置

在Windows 系统上, 需要下载 Maven 的 zip 文件，并将其解压到你想安装的目录，并配置 Windows 环境变量。

所需工具 ：

* JDK 1.8
* Maven 3.3.3
* Windows 7

具体安装配置方法[传送门](https://www.yiibai.com/maven/maven_environment_setup.html)

在mac系统下，可以在终端执行`brew install maven` 前提是安装了[homebrew](https://brew.sh/)工具

安装成功可以在终端测试
>
>`mvn -v`
>
输出
```

Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-18T02:33:14+08:00)
Maven home: /usr/local/Cellar/maven/3.5.4/libexec
Java version: 1.8.0_181, vendor: Oracle Corporation, runtime: /Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.13.6", arch: "x86_64", family: "mac"


```

将一个war包安装在本地mvn仓库可执行如下命令

`mvn install:install-file -Dfile=/Users/anry/fedir/dist.war   -DgroupId=com.yonyou.iuap -DartifactId=iuap_hello_fe  -Dversion=2.1.4-fe-SNAPSHOT -Dpackaging=war`

将一个war包安装推送到远程mvn仓库可执行如下命令

`mvn deploy:deploy-file  -Dfile=/Users/anry/fedir/dist.war   -DgroupId=com.yonyou.iuap -DartifactId=iuap_hello_fe  -Dversion=2.1.4-fe-SNAPSHOT -Dpackaging=war  -DrepositoryId=iUAP-Snapshots -Durl=http://mavendemorepo.com/nexus/content/repositories/iUAP-Snapshots/`

如果推送失败，可以根据错误提示，查找原因，一般情况下，检查maven conf的配置文件，前端推送的maven war包 ，会在pom.xml被配置依赖，启动maven项目会将前端依赖解压在target下，保证页面可被访问。启动web服务，会根据配置，将资源copy到服务器部署webapp的目录

#### 3.2.2Maven POM

POM代表项目对象模型。它是 Maven 中工作的基本单位，这是一个 XML 文件。它始终保存在该项目基本目录中的 pom.xml 文件。
POM 包含的项目是使用 Maven 来构建的，它用来包含各种配置信息。
POM 也包含了目标和插件。在执行任务或目标时，Maven 会使用当前目录中的 POM。它读取POM得到所需要的配置信息，然后执行目标。部分的配置可以在 POM 使用如下：

* project dependencies
* plugins
* goals
* build profiles
* project version
* developers
* mailing list

创建一个POM之前，应该要先决定项目组(groupId)，它的名字(artifactId)和版本，因为这些属性在项目仓库是唯一标识的。

一个 _pom.xml_ 示例

```

<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.yiibai.project-group</groupId>
   <artifactId>project</artifactId>
   <version>1.0</version>
<project>

```

#### 3.2.3Maven其他知识

有部分前端的同学对maven不了解，可以访问[教程](https://www.yiibai.com/maven/)获取相关知识,以及运行原理

* Maven本地资源库
* Maven 构建生命周期
* Maven Web应用
* ...

### 3.3maven自动化推送

将前端源码编译后打包压缩，生成一个war文件，然后执行 _mvn install_ 可将war部署在本地仓库，_mvn deploy_ 可将资源推送到远程仓库

#### 3.3.1gulp deploy任务

在 _gulpfile.js_ 引入依赖文件,增加打包推送的任务，代码如下：

```

var gulp = require('gulp');

var uposs = require('./uposs');
var change = require('./change');
var cfg = require('./conf/config');
var maven = require('./deploymvn');

//发布新版本前修改changelogs 记录改动情况 以及日期
var changelogs = cfg.changelog || [];

//是否生产环境判断
var isProd = process.env.env === 'prod';

// output dist.war
gulp.task("package", function () {
    gulp.src(['dist/**'])
    .pipe(zip('dist.war'))
    .pipe(gulp.dest('./'));
});

//发布maven
gulp.task('deploy', ['package'], function () {
    maven.install();//本地安装
    maven.deploy(); //远程推送
});


```

deploymvn.js 定义如下：

需要配置publishConfig，

//publishConfig定义了执行mvn install和mvn deploy 的参数示例如下：
```
    // publishConfig: {
    //     command: "mvn",
    //     repositoryId: "iUAP-Snapshots",
    //     repositoryURL: "http:/yourdeploymaven.url/nexus/content/repositories/iUAP-Snapshots/",
    //     artifactId: "iuap_your_fe",
    //     groupId: "com.yonyou.iuap",
    //     version: "2.1.4-yourfe-SNAPSHOT"
    * // },

```    
在deploymvn模块中使用了node fs模块和child_process模块
> fs模块提供文件系统的api，处理文件的读写操作
> child_process模块，可以创建子进程，child_process.exec方法可以创建异步进程

```

var publishConfig = require('./conf/config').publishConfig;


var fs = require('fs');
module.exports = {
    install: function () {
        if (!publishConfig) {
            console.console.error("can't find publishConfig in config.js");
        }
        var targetPath = fs.realpathSync('.');
        var installCommandStr = publishConfig.command + " install:install-file -Dfile=" + targetPath + "/dist.war   -DgroupId=" + publishConfig.groupId + " -DartifactId=" + publishConfig.artifactId + "  -Dversion=" + publishConfig.version + " -Dpackaging=war";
        console.log(installCommandStr);
        var process = require('child_process');
        var installWarProcess = process.exec(installCommandStr, function (err, stdout, stderr) {
            if (err) {
                console.log('install war error:' + stderr);
            }
        });
        installWarProcess.stdout.on('data', function (data) {
            console.info(data);
        });
        installWarProcess.on('exit', function (data) {
            console.info('install war success');
        })

    },
    deploy: function () {
        if (!publishConfig) {
            console.console.error("can't find publishConfig in config.js");
        }
        var process = require('child_process');
        var targetPath = fs.realpathSync('.');
        var publishCommandStr = publishConfig.command + " deploy:deploy-file  -Dfile=" + targetPath + "/dist.war   -DgroupId=" + publishConfig.groupId + " -DartifactId=" + publishConfig.artifactId + "  -Dversion=" + publishConfig.version + " -Dpackaging=war  -DrepositoryId=" + publishConfig.repositoryId + " -Durl=" + publishConfig.repositoryURL;
        console.info(publishCommandStr);
        var publishWarProcess = process.exec(publishCommandStr, function (err, stdout, stderr) {
            if (err) {
                console.log('publish war error:' + stderr);
            }
        });

        publishWarProcess.stdout.on('data', function (data) {
            console.info(data);
        });
        publishWarProcess.on('exit', function (data) {
            console.info('publish  war success');
        });
    }
}


```
gulpfile任务编写完成，接下测试一下

#### 3.3.2本地测试
>
>在终端执行
>
>`gulp deploy`
>
![gulpdeploy](https://github.com/Anryliu/anryblog/blob/master/1534920844434.jpg?raw=true)
在deploymvn加了一些consoleinfo的信息，会输出在控制台上，也会看到publish  war success的输出，说明war包推送成功，我们可在本地和远程mvn仓库检查war包是否存在正确

#### 3.3.3为什么要发布一个前端的war包? :beers:

在前后端分离的开发模式中，前端推送maven项目依赖的war包，正好解决了web应用中，前端资源依赖的问题，也便于前端项目版本迭代升级，可以做到个性化输出。

当然可以把前端所有的编译打包任务，都交由maven的插件去处理，前端开发人员只需要在gitlab等托管平台上提交代码即可，自动化部署的话题在下一次的分享中可以详细说明下。

## 四.推送CDN资源

CDN是构建在网络之上的内容分发网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。CDN的关键技术主要有内容存储和分发技术。

依托CDN技术，可以提升站点的访问速度和稳定性，保证不同网络中的用户都能得到良好的访问质量。带宽优化，以及预防一些网络入侵，降低DDOS攻击对站点的影响。

CDN将源站内容分发至最接近用户的节点，使用户可就近取得所需内容，提高用户访问的响应速度和成功率。解决因分布、带宽、服务器性能带来的访问延迟问题，适用于站点加速、点播、直播等场景。

这里以使用阿里云存储cdn资源的案例实现cdn自动化上传

借用一张图
![CDN应用](https://img.alicdn.com/tps/TB1JjgZOFXXXXX6XXXXXXXXXXXX-1530-1140.png)

### 4.1安装依赖，增加gulp任务


>
>`npm install ora ali-oss co co-request  —save-dev`
>
>
增加deploycdn.js,配置阿里云授权id，密钥仓库，地区等参数如下：
```
    //ossconfig配置
    // ossconfig: {
    //     accessKeyId: 'youraccessKeyId',
    //     accessKeySecret: 'youraccessKeySecret',
    //     bucket: 'your bucket',
    //     region: 'oss-cn-beijing',
    // },

```

尝试了几个其他的npm oss的库，不是很好用，所以借鉴实现了deploycdn模块，内容如下：
```

var fs = require('fs');
var co = require('co'); //promises,实现异步任务
var path = require('path'); 
var oss = require('ali-oss');//ali-oss-js-sdk
var ossconfig = require('./conf/config').ossconfig;

var store = oss(ossconfig);

module.exports= function(date){
    date  =  typeof date ==='string'? date:'dist';
    var root = path.resolve(__dirname, './'+date);
    var files = [];
    //递归取出所有文件夹下所有文件的路径
    function readDirSync(p) {
        var pa = fs.readdirSync(p);
        pa.forEach((e) => {
            var cur_path = `${p}/${e}`;
            var info = fs.statSync(cur_path);
            if (info.isDirectory()) {
                readDirSync(cur_path);
            } else {
                files.push(cur_path);
            }
        });
    }
    readDirSync(root);
    co(function* () {
        //遍历文件
        for (let index = 0; index < files.length; index += 1) {
            var e = files[index];
            var result = yield store.put(e.replace(root, 'pro/baseData/'+date), e);
            //提交文件到oss，这里要注意，阿里云不需要创建新文件夹，只有有路径，没有文件夹会自动创建
            // console.log(result);
        }
    });

}

```

在gulpfile.js增加任务

```


var uposs = require('./deploycdn');
var change = require('./change');
var cfg = require('./conf/config');

//发布新版本前修改changelogs 记录改动情况 以及日期
var changelogs = cfg.changelog;

var isProd = process.env.env === 'prod';

//文件上传到ali oss
gulp.task('uposs', function () {
    if (isProd) {
        uposs(date);
        //write changeLog into changelog.md
        change(date, changelogs || []);
    }
});



```

### 4.2uposs任务测试
>
在终端执行
>
>`gulp uposs`
>
如图示

![gulp uposs](https://github.com/Anryliu/anryblog/blob/master/1534920816208.jpg?raw=true)

## 五.gulp任务改进及工程构建

以上两部分内容分别实现了发布war包和上传cdn资源，但实际上我们每次都要在终端输入 _gulp deploy_ 和 _gulp uposs_ 也是挺闹心，为此我们在gulpfile.js配置一个新的build任务，build任务可包括，源码编译压缩，文件合并，多语翻译，cdn上传，deploy war包等集合任务，构建生产环境只需要执行gulp build，为了区分开发环境在node的process.env增加env的属性用来区分生产环境和测试开发环境，然后配置在package.json中，具体示例：

```

var isProd = process.env.env === 'prod';

gulp.task('build', function (callback) {
    if (isProd) {
        sequence('clean', 'less:dist', ['minicss', 'minijs','replace'], 'i18n',['deploy', 'uposs'], callback);
    } else {
        sequence('clean', 'less:dist', 'watch'], callback);
    }
});

```

在package.json中增加

```

"scripts": {
    "dev": "env=develop gulp build",
    "build": "env=prod gulp build",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  
```

>
>开发调试时，运行
>
>npm run dev
>
>发布线上生产环境时，运行
>
>
>npm run build
>

也是一壶爽的事情... :beers:

### 5.1构建过程

源码 -> | 编译 -> | 产出（线上线下）
---------|----------|---------
scss less es6  ts jsx 等文件| node环境 global对象 gulp webpack工具 执行编译任务 | mincss minjs html等

## 六.尾记

最近有前端同学正在做前端项目构建，发布cdn，发布maven，尝试前后端分离，解决部署的问题，前端的同学可以尝试着用一下，有更好的意见和建议欢迎分享提问，大家一起交流。

一些基本的概念和使用方法没有做详细介绍，需要学习的话，利用下网络资源,比如nodejsAPI fs模块 process模块 gulpplugins gulp高阶使用等等

>### 感谢郭(永锋)老师提出修改意见

附赠一张我的手写体

![随缘](https://github.com/Anryliu/shufalianxi/raw/master/shufa/WechatIMG3.jpeg)
