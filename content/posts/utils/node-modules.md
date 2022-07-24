---
title: "批量清理node_modules"
date: 2021-12-05T11:30:03+00:00
tags: ["npm","nodejs","scripts"]
series: ["实用小脚本"]
categories: ["脚本"]
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

```
find . -name "node_modules" -type d -prune | xargs du -chs
```

![image.png](https://b3logfile.com/file/2021/05/solo-fetchupload-3818493748119197664-9ee7eb4c.png)

```
find . -name "node_modules" -type d -prune -exec rm -rf '{}' +
```

