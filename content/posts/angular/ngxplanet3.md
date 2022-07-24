---
title: "关于微前端实现原理与ngx-planet(三)"
date: 2021-12-05T11:30:03+00:00
tags: ["angular"]
series: ["ngx-planet微前端"]
categories: ["微前端"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS:  false
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---

# 1.为什么要服务端渲染

因为公司后端服务在k8s上，是分布式的微服务，之前端全部打包部署在了物理机器(虚拟机)nginx上,如果通过helm做应用商店的话，nginx前端这部分无法处理，包括灰度部署，CI等，全部都只能做到接口级别的处理，并不能连带静态资源文件一起处理，所以基于分布式的前端整改迫在眉睫。

偶然发现了ngx-planet，整篇文章基于前几篇文章，可以看之前的几篇文章。

# 2.如何基于ngx-palnet进行服务端渲染

## 2.0 有路由前缀的情况下服务端渲染ngx-planet如何改进

![image.png](https://b3logfile.com/file/2021/02/image-11f71cd7.png)

在 `start`函数中监听路由阶段，其中的 `startWith`过滤路由要把 `location.pathname`修改好，要将前缀去除掉，至于如何动态去除不写死，仁者见仁智者见智。

## 2.1 首先，准备打包好的静态文件

将 `build`命令修改，注意 `--deploy-url`的意思是，打包完成后，静态资源路径是什么

```
    "build": "ng build --prod --deploy-url=/static/star-universe/ --base-href=/star-universe",

```

![image.png](https://b3logfile.com/file/2021/02/image-2baa6815.png)

打包后的index.html如图，静态资源路径变成了 `/static/star-universe`前缀，这里和 `base-href`不能冲突，否则在渲染时有死循环问题

其次 `base-href`就是项目访问路径前缀

然后修改 `angular.json`中的 `build`放到你的静态资源目录，这样在执行 `npm run build`后静态资源自动放入后台项目中的静态目录中

![image.png](https://b3logfile.com/file/2021/02/image-93ae712c.png)

## 2.2 准备Portal后台项目

我的项目基于SpringBoot自动生成，项目结构如下

![image.png](https://b3logfile.com/file/2021/02/image-6996d199.png)

以下是用到的maven配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.3</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.yunzainfo.cloud</groupId>
    <artifactId>star-universe</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>star-universe</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

一个Controller负责重定向路由到 `index.html`

```java
package com.yunzainfo.cloud.staruniverse.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;

@Controller
public class StaticController {

    @RequestMapping(value = {"/star-universe", "/star-universe/**"})
    public String index(HttpServletRequest request) {
        System.out.println(request.getRequestURI());
        return "index";
    }

}

```

portal的静态资源处理，通过实现 `WebMvcConfigurer`的 `addResourceHandlers`函数,注意函数传参数的静态资源路径谓 `/static/star-universe/**`要和 `--deploy-url`对应起来，保证 `index.html`中的静态资源被正确加载

```java
package com.yunzainfo.cloud.staruniverse.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/star-universe/**").addResourceLocations("classpath:/static/");
    }
}

```

然后是portal的applicaiton.yml，给一个端口吧，这里我选择的是3000,和proxy.config中的相同，至于静态资源路径为什么是 `/static`也是为了和注册应用时的路径保持一致

```yml
server:
  port: 3000

spring:
  thymeleaf:
    cache: false
    prefix: classpath:/static/


```

关于 `app.component`中的应用注册,`resourcePathPrefix`和 `manifest`全都加入应用路径完成远程注册

```json
[
  {
    "name": "star-dust",
    "hostParent": "#wormhole",
    "hostClass": "star-dust",
    "routerPathPrefix": "/star-dust",
    "stylePrefix": "star-dust",
    "resourcePathPrefix": "http://localhost:3001/static/star-dust/",
    "loadSerial": true,
    "preload": false,
    "switchMode": "coexist",
    "scripts": [
      "main.js"
    ],
    "styles": [
      "styles.css"
    ],
    "manifest": "http://localhost:3001/static/star-dust/manifest.json"
  }
]

```

## 2.3 准备子项目前台项目

同样修改 `build`和 `angular.json` 然后打包

## 2.4 准备子项目后台项目

这里唯一和portal项目不同的时，要加入跨域访问允许，因为portal加载js/css的时候是远程加载的。

```java
package com.yunzainfo.cloud.stardust.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.*;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/star-dust/**").addResourceLocations("classpath:/static/");
    }

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowCredentials(true)
                .allowedOrigins("http://localhost:3000", "http://localhost:3001", "http://localhost:3002", "http://localhost:3003")
                .allowedMethods("POST", "GET", "PUT", "OPTIONS", "DELETE")
                .maxAge(3600)
                .allowCredentials(true);

    }
}

```

其余和portal一致，只是路径不一样


# 3.看效果

![image.png](https://b3logfile.com/file/2021/02/image-5da8d240.png)

![image.png](https://b3logfile.com/file/2021/02/image-10f2850a.png)

这样，静态资源全都被远程请求过来了


# 4.这种远程技术可以运用到什么地方

首先组件共享的好处不说了，结合了后端微服务的 `真正的`前端微服务而不是依托于 `nginx`静态部署的前端微服务

可以优化开发流程，可以做灰度部署了，可以完善CI了等等，好处也有，坏处也有。有利有弊，看各自需要吧。