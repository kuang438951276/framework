# framework
# 一、建立project，做好基础配置

## 1.在gradle.properties配置好module中需要公用的属性，方便维护，如：

```
#<--------------------------------------版本配置start-------------------------------------->
# 这个值一般跟你的AndroidStudio版本号一致
localGradlePluginVersion=2.3.1

# 为自动化出包配置(因为每个开发的电脑坏境不一致)
localBuildToolsVersion=26.0.0

# 本地maven私库地址，用户名和密码
localMavenUrlRelease=http://192.168.8.208:9999/nexus/content/repositories/topbandlib/
localMavenUrlSnapshot=http://192.168.8.208:9999/nexus/content/repositories/topbandlib-snapshot/
localMavenUser=kuangyong
localMavenPwd=123456

#<--------------------------------------版本配置end-------------------------------------->


#<--------------------------------------项目配置start-------------------------------------->
#commonlib是否独立运行
isCommonLib=false

#<--------------------------------------项目配置end-------------------------------------->
```
## 2.在build.gradle文件中配置相关属性，方便module中统一引用，如：

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath "com.android.tools.build:gradle:$localGradlePluginVersion"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
        mavenCentral()
        //支持arr包
        flatDir {
            dirs 'libs'
        }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

ext{
    buildToolsVersion=localBuildToolsVersion
    compileSdkVersion=25
    minSdkVersion=16
    targetSdkVersion=25
    versionCode=1
    versionName='1.0.0'

    okhttp3Version='3.8.1'
    loggingInterceptorVersion='3.4.1'
    gsonVersion='2.7'
}

```

# 二、建立module，配置maven本地私服库方便引用，如：
```
if (isCommonLib.toBoolean()) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}
apply plugin: 'maven'
android {

    compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion
    defaultConfig {
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode rootProject.ext.versionCode
        versionName rootProject.ext.versionName
    }
}
/**
 * 上传aar包脚本
 */
uploadArchives {
    repositories {
        mavenDeployer {
            snapshotRepository(url: project.localMavenUrlSnapshot) {
                authentication(userName: project.localMavenUser, password:  project.localMavenPwd)
            }

            /**
             * 默认路径以及用户、密码配置
             */
            repository(url: project.localMavenUrlRelease) {
                authentication(userName:  project.localMavenUser, password: project.localMavenPwd)
            }

            /**
             * 版本号及分组
             */
            pom.project {
                version '1.0.0'
                artifactId 'commonlib'
                groupId 'com.topband.lib'
                packaging 'aar'
                description 'dependences lib'
            }
        }
    }
}

artifacts {
    archives file('commonlib.aar')
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:recyclerview-v7:26.+'
    compile 'com.android.support:appcompat-v7:26.+'
}

```

#本地maven私服库

# Android 项目部署之Nexus私服搭建和应用

## 一、概述
 &emsp;Nexus是一个基于maven的仓库管理的社区项目.主要的使用场景就是可以在局域网搭建一个maven私服,用来部署第三方公共构件或者作为远程仓库在该局域网的一个代理.简单举几个例子就是:
### 1.第三方Jar包可以放在nexus上,项目可以直接通过Url和路径配置直接引用.方便进行统一管理.
### 2.同时有多个项目在开发的时候,一些共用基础模块可以单独抽取到nexus上,需要用的项目直接从nexus上拉取就行
### 3.一些封闭开发的过程中开发机是不能上公网的,所以连接central repository和下载jar就比较麻烦,这时就可以用nexus搭建起来一个介于公网和局域网之间的桥梁.
## 二、搭建
### 1.下载&配置

&emsp;这里使用的是Nexus OSS开源版,官网下载地址:http://www.sonatype.org/nexus/go/ 

&emsp;把压缩包解压之后进入bin文件夹就可以运行cmd命令来启动nexus,通过查看bin/nexus脚本发现可以修改脚本来适配自己的需求,例如修改Nexus的root路径,如果需要以root身份来启动Nexus就需要设置RUN_AS_USER=root,设置app名字和登陆名字等.也可以去conf/nexus.properties文件修改端口之类的信息. 

&emsp;接下来直接运行Nexus脚本,进入解压之后的文件夹的bin目录下，使用cmd命令nexus start启动服务
