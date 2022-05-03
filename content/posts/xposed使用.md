---
title: "Xposed使用"
date: 2022-05-03T19:59:29+08:00
draft: false
tags: ['xposed', 'android']
---

简单介绍一下编写xposed的过程

<!--more-->

## 导入Xposed

```
dependencies {
    compileOnly 'de.robv.android.xposed:api:82'
    compileOnly 'de.robv.android.xposed:api:82:sources'
}
```

> 注意要在`settings.gradle`里面加入xposed的maven库

```
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url 'https://api.xposed.info/'}
    }
}
```



## 在`AndroidManifest.xml`中导入配置信息

需要在`AndroidManifest.xml`中加入三个配置

```xml
<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Demo_xposed" >
        <meta-data
            android:name="xposedmodule"
            android:value="true" />
        <meta-data
            android:name="xposeddescription"
            android:value="Easy example which makes the status bar clock red and adds a smiley" />
        <meta-data
            android:name="xposedminversion"
            android:value="42+" />
    </application>
```

+ `xposedmodule` 是否是xposed模块
+ `xposeddescription` 指定模块的描述信息
+ `xposedminversion` 指定其xposed的版本



## 在assets文件夹中写入`xposed_init`文件

在assets中新建一个文本文件名字叫做`xposed_init`

里面填入需要xposed hook的主类，比如: `com.funny.demo_xposed.Main`

里面简单写一个Java代码

```java
public class HelloXp implements IXposedHookLoadPackage {
    private final String TAG = "demo_xposed";
    @Override
    public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
    		Log.d("xposed_demo", "Hook Success!");
    }
}
```
