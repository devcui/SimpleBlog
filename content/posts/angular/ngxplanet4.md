---
title: "关于微前端实现原理与ngx-planet(四)"
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

# 客制化

由于公司和公司的业务不同，所以在`ngx-planet`的基础上，需要作出一些针对于业务的拓展

目前分出四个基础项目

`@yunzai/stars`:封装了`用户认证`,`元素权限`,`i18n`等系统初始化信息的内容,需要发包的,意为繁星.

`star-universe`: `portal`项目,所有子前端项目的入口,意为宇宙.

`star-dust`:一些可能会通用的`components`都放到这个项目里,统一管理,意为星尘.

`star-uranus`:模版项目,意为天王星.

# 开发环境

![image.png](https://b3logfile.com/file/2021/02/image-e1a4e4ff.png)

可以看到桌面共有四个编辑器，开发时需要跑起其中三个项目.
`star-universe`,`star-dust`,`star-uranus`

# @yunzai/stars

因为是基础包，所以扩展了哪些功能点，先做一个介绍，因为发包，所以planet前缀都被我改名成为了star前缀，针对于ngx-planet加载过程请看前一篇<a href="https://blog.eiyouhe.com/articles/2021/01/22/1611298349966.html">link start</a>。

接下来看项目结构

![image.png](https://b3logfile.com/file/2021/02/image-83e2c9df.png)

首先基于`ngx-planet`加入了`auth`,`config`,`i18n`,`providers`,`stomp`,`token`,`user`这些文件夹

## module

针对于Module做出了以下改良
![image.png](https://b3logfile.com/file/2021/02/image-75c8a809.png)

加入了TranslateModule来写i18n
加入了`Inject`的静态配置类`StarConfig`

```ts
/*
 * @Author: ferried
 * @Email: harlancui@outlook.com
 * @Date: 2021-02-01 08:59:04
 * @LastEditTime: 2021-02-03 09:37:51
 * @LastEditors: ferried
 * @Description: Basic description
 * @FilePath: /stars-package/projects/stars/src/star-config.ts
 * @LICENSE: Apache-2.0
 */

import { InjectionToken } from "@angular/core";
import { InjectableRxStompConfig } from "@stomp/ng2-stompjs";

/**
 * 配置接口
 */
export interface StarConfig {
  /**
   * 网关
   */
  gateway: string;
  /**
   * cas
   */
  cas: string;
  /**
   * 忽略地址
   */
  ignores: Array<string>;
  /**
   * 开发模式
   */
  dev?: {
    // 用户名
    username: string;
    // 密码
    password: string;
    // 是否开启网络
    network: boolean;
  };
  /**
   * 长连接
   */
  stomp?: InjectableRxStompConfig;
}

/**
 * inject token
 */
export const STAR_CONFIG = new InjectionToken<StarConfig>("STAR_CONFIG");
```

这些配置在包中可以被`inject`到然后使用，方便开发，但是只有`portal`项目注入了配置，如何在其他子应用中使用呢，这里使用了`APP_INITIALIZER`

![image.png](https://b3logfile.com/file/2021/02/image-cd204afa.png)

提供了一个初始化的工程函数，返回一个`Promise`给angular,配置到子应用中的`providers`中用来初始化各个子应用的`config`
![image.png](https://b3logfile.com/file/2021/02/image-06d0ed29.png)

具体如何实现呢

## 扩展portalApplication

![image.png](https://b3logfile.com/file/2021/02/image-8ff2e514.png)

可以看到，`portalApplication` 被我扩展出了6个其他的属性，这些属性在`PortalApplication`被创建时由`portal的context使用inject`装入,这里区分`context`很重要

![image.png](https://b3logfile.com/file/2021/02/image-427804f3.png)

并放到了全局`window`下，那么子应用使用的时候就可以直接从`window`下获取了，但是比较怪，想要通过`构造器`注入使用应该怎么办呢

可以单独提供一个ConfigService

![image.png](https://b3logfile.com/file/2021/02/image-e77ca45f.png)

![image.png](https://b3logfile.com/file/2021/02/image-220fd262.png)

这样子应用使用直接注入的方式就可以拿到`config`配置

![image.png](https://b3logfile.com/file/2021/02/image-22857386.png)

这样就做到了通过`window`和`portal`流转，`service`共享的机制

# portal provider和star provider

## portal

![image.png](https://b3logfile.com/file/2021/02/image-4ecc44fb.png)

这是整体`portal`的`provider`

```ts
export function PortalStartupServiceFactory(
  authStartupService: AuthStartupService,
  i18nStartupService: I18nStartupService,
  configStartupService: ConfigStartupService,
  portalInfoStartupService: PortalInfoStartupService
): Function {
  return () => {
    return new Promise((resolve, reject) => {
// 先将config共享
      configStartupService.load().then(() => {
// portal进行用户认证决定跳入登录还是进入子应用
        authStartupService.load().then(() => {
// 初始化一些logo,footer等乱七八糟的信息
          portalInfoStartupService.load().then(() => {
// 初始化i18n远程拉过来的语言列表等
            i18nStartupService.load().then(() => {
              resolve(true);
            });
          });
        });
      });
    });
  };
}
```

使用时，在`star-universe`中直接加入即可
![image.png](https://b3logfile.com/file/2021/02/image-2e60a54d.png)

如图可见，在使用`YunzaiStarsModule`的时候注入了一些配置信息，`config`就是通过这些从`portal`注入的信息去分享的`service`,然后`providers`中也加载了一些`APPINIT_PROVIDES`,剩下的`HTTP_INTERCEPTORS`是两个拦截器，分别在httpHeaders中加入了一些配置信息

## star

![image.png](https://b3logfile.com/file/2021/02/image-5ae2f5c4.png)

上图是`star`的初始化操作，只共享了config和i18n的信息。
这里`i18n`是必须要加入的，使用`translate pipe`时是从当前的`module`中提取的`translateService`去进行翻译的,所以子应用不初始化`i18n`会导致，子应用中的模块`module`没有加载`i18n的语言列表`从而`translate`管道不生效,和`config`共享机制相同

![image.png](https://b3logfile.com/file/2021/02/image-4bb11a62.png)

上图为`子应用的`初始化`Providers`，和`portal`不同的是，无需注入`额外的配置了`因为通过`providers`和`window`流转到子应用的`service`中了。

# 关于集成ng-zorro-antd

正常ng-add进`ng-zorro-antd`包，然后修改`webpack`匹配的`scss`为`less`,其中有一个bug,无论使用`ngx-planet`的scss或是`ng-zorro-antd`的`less`都会出现,已经提了<a href="https://github.com/worktile/ngx-planet/issues/170">issues</a>

目前的解决方法是建立一个`component.less/scss`然后引入全局的`style.less/css'
![image.png](https://b3logfile.com/file/2021/02/image-e74be424.png)

![image.png](https://b3logfile.com/file/2021/02/image-e954bd78.png)

## portal的extra-webpack-config

![image.png](https://b3logfile.com/file/2021/02/image-129e9ad2.png)

## 子应用的extra-webpack-config

![image.png](https://b3logfile.com/file/2021/02/image-5cd27563.png)

# portal作为layout

以下为portal的路由
![image.png](https://b3logfile.com/file/2021/02/image-894a6902.png)

在layout中加入挂载点

![image.png](https://b3logfile.com/file/2021/02/image-191a46f1.png)

就可以做到layout在portal中，其余页面分布在子应用中了

# 事件注册，组件注册扩展

![image.png](https://b3logfile.com/file/2021/02/image-9f744958.png)

事件注册加了一层`应用`与`事件`关联的map

这样就可以知道哪些应用放出了哪些事件

![image.png](https://b3logfile.com/file/2021/02/image-2b0e90d0.png)

组件注册加了一层`应用`与`组件`的关联map

这样就可以知道哪些应用放出了哪些组件

# stomp长链接再配置

由于portal是整个应用群的入口，所以是常驻的，stomp模块直接加在portal里注册一个事件即可全应用通用，但是在部署的时候会有域名/无域名/有https/无https，开发人员不想每次都重新打包怎么办呢

我们修改注入到`RxStompService`的`Config`
![image.png](https://b3logfile.com/file/2021/02/image-a1123986.png)

提供我们自己的Config

```ts
/*
 * @Author: ferried
 * @Email: harlancui@outlook.com
 * @Date: 2021-02-03 09:35:56
 * @LastEditTime: 2021-02-03 09:38:52
 * @LastEditors: ferried
 * @Description: Basic description
 * @FilePath: /stars-package/projects/stars/src/stomp/stomp.ts
 * @LICENSE: Apache-2.0
 */
import { RxStompConfig } from "@stomp/rx-stomp";
import { StarConfig } from "../config/star-config";

export function YzStompConfigFactory(starConfig: StarConfig): RxStompConfig {
  const stomp: RxStompConfig = new RxStompConfig();
  if (starConfig.stomp) {
    let brokerUrl = null;

    if (document.location.protocol === "https:" && starConfig.stomp.brokerURL) {
      brokerUrl = `wss://${window.location.host}${starConfig.stomp.brokerURL}`;
    }
    if (document.location.protocol === "http:" && starConfig.stomp.brokerURL) {
      brokerUrl = `ws://${window.location.host}${starConfig.stomp.brokerURL}`;
    }

    if (starConfig.stomp && starConfig.stomp.brokerURL) {
      if (
        starConfig.stomp.brokerURL.split(":")[0] === "wss" ||
        starConfig.stomp.brokerURL.split(":")[0] === "ws"
      ) {
        brokerUrl = starConfig.stomp.brokerURL;
      }
    }

    if (brokerUrl) {
      stomp.brokerURL = brokerUrl;
    }

    if (starConfig.stomp.connectHeaders && starConfig.stomp.connectHeaders) {
      stomp.connectHeaders = starConfig.stomp.connectHeaders;
    }

    if (starConfig.stomp.heartbeatIncoming) {
      stomp.heartbeatIncoming = starConfig.stomp.heartbeatIncoming;
    }
    if (starConfig.stomp.heartbeatOutgoing) {
      stomp.heartbeatOutgoing = starConfig.stomp.heartbeatOutgoing;
    }
    if (starConfig.stomp.reconnectDelay) {
      stomp.reconnectDelay = starConfig.stomp.reconnectDelay;
    }
  }
  return stomp;
}
```

# 关于cli

等到项目完成后要加入schematic然后通过`ng add @yunzai/stars`来自动初始化项目，目前`star-uranus`模版没调好所以还无法做

具体如何做可以通过`ng-zorro-antd`来实现一下
<a href="https://blog.eiyouhe.com/articles/2019/08/19/1566177948951.html">cli</a>