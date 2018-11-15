---
title:        Add release to sentry
date: 2018/11/12 21:24:39
categories:
  - java
tags:
  - java
  - sentry
---

[Sentry](https://sentry.io) 是一个开源的，用来帮助开发监控异常和跟进错误修复的服务。

Java 集成 Sentry 十分地方便，很方便地就可以实现异常的监控。如果我们想更加精细化地区分某个错误是在哪个版本中出现的，需要我们额外将 release 信息告知 Sentry。具体怎么操作呢？

## 1. 添加`SentryConfig`配置类，将相关信息注入到 Sentry

```java
@Configration
public class SentryConfig implements InitializingBean {

    @Value("${application.host.name:localhost}")
    private String hostName;

    @Value("${application.module.name}")
    private String serviceName;

    @Value("${spring.profiles.active}")
    private String activeProfile;

    @Value("${release.version:local}")
    private String releaseVersion;
    
    // 将 sentry 的配置放到 application.properties 中，
    // 可以根据环境配置到不同 sentry 服务地址
    @Value("${sentry.dsn}")
    private String sentryDsn;

    @Value("${sentry.timeout}")
    private String sentryTimeout;

    @Override
    public void afterPropertiesSet() throws Exception {
        MDC.put("Host", hostName);
        MDC.put("Module", serviceName);
        MDC.put("Profile", activeProfile);

        System.setProperty("sentry.environment", activeProfile);
        System.setProperty("sentry.release", releaseVersion);

        if (StringUtils.isNotBlank(sentryDsn)) {
            System.setProperty("sentry.dsn", sentryDsn);
            System.setProperty("sentry.timeout", sentryTimeout);
        }
    }
}
```

我们需要解决 `release.version` 从哪里获取，简单一些可以把 `project.version`在 build 的时候，写入到`application.properties`中。

```groovy
task createProperties(dependsOn: processResources) {
    doLast {
        def releaseVersion = 'release.version=' + project.version.toString()
        def file = new File("$buildDir/resources/main/application.properties")
        if (file.exists()) {
            file.append(releaseVersion)
        }

        file = new File("$projectDir/resources/main/application.properties")
        if (file.exists()) {
            file.append(releaseVersion)
        }
    }
}

classes {
    dependsOn createProperties
}
```

我们可以继续完善：在版本信息的基础上把 Git 的 Commit ID 也加上，可以这样做：

## 方法 1. 添加[gradle-git-version](https://github.com/palantir/gradle-git-version)

```groovy
buildscript {    
    dependencies {
        classpath('gradle.plugin.com.palantir.gradle.gitversion:gradle-git-version:0.11.0')
    }
}

apply plugin: 'com.palantir.git-version'
```

添加这个插件后，我们可以调用`versionDetails()`获取当前项目的一些信息：

    def details = versionDetails()
    details.lastTag
    details.commitDistance
    details.gitHash
    details.gitHashFull // full 40-character Git commit hash
    details.branchName // is null if the repository in detached HEAD mode
    details.isCleanTag

里面有我们需要的`gitHash`，所以我们上面的脚本就可以修改为:

```groovy
task createProperties(dependsOn: processResources) {
    doLast {
        def releaseVersion = 'release.version=' + project.version.toString() = '-' + versionDetails().gitHash
        def file = new File("$buildDir/resources/main/application.properties")
        if (file.exists()) {
            file.append(releaseVersion)
        }

        file = new File("$projectDir/resources/main/application.properties")
        if (file.exists()) {
            file.append(releaseVersion)
        }
    }
}

classes {
    dependsOn createProperties
}
```

上面的实现需要添加较多地配置，我们可以使用另外一个 gradle 插件[gradle-git-properties](https://github.com/n0mer/gradle-git-properties) 达到类似的效果：
## 方法 2：使用 gradle-git-properties 插件

1. 添加依赖：

```groovy
buildscript {
    dependencies {
        classpath("gradle.plugin.com.gorylenko.gradle-git-properties:gradle-git-properties:1.5.1")
    }
}

apply plugin: 'com.gorylenko.gradle-git-properties'
```

gradle-git-properties 会添加一个**generateGitProperties**的任务，执行之后生成`git.properties`；它相较于 git-version 提供的内容更多：

    git.branch
    git.build.host
    git.build.time
    git.build.user.email
    git.build.user.name
    git.build.version
    git.closest.tag.commit.count
    git.closest.tag.name
    git.commit.id
    git.commit.id.abbrev
    git.commit.id.describe
    git.commit.message.full
    git.commit.message.short
    git.commit.time
    git.commit.user.email
    git.commit.user.name
    git.dirty
    git.remote.origin.url
    git.tags
    git.total.commit.count

我们需要的是`git.commit.id.abbrev`，为了输出`release.version`我们需要添加一行配置：

```java
// build.gradle
gitProperties {
    customProperty 'release.version', { project.version + "-" + it.head().abbreviatedId }
}
```

因为它生成的 properties 文件默认不会加载，需要在`SentryConfig`上添加`@PropertySource`注解，加载它:
```java
@PropertySource(value = "classpath:git.properties", ignoreResourceNotFound = true)
```

在程序抛出异常后，我们就可以在 Sentry 中看到异常信息已经包含了应用的版本及 commitId 信息了：
{% asset_img 15420289952643.jpg %}
