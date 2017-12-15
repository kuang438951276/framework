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

maven本地私服库 https://github.com/kuang438951276/maven
