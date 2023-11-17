---
title: "以 GitHub 作为图床创建图片 Service API"
date: 2023-11-17T15:35:36+08:00
draft: false
comment: true
description: "本文介绍如何基于 GitHub 为图床，通过 API 随机返回可用的图片地址。"
---

本文介绍如何基于 GitHub 为图床，通过 API 随机返回可用的图片地址。

## 前言

常用的桌面壁纸、终端背景图片，亦或是博客背景或文章封面，这些都离不开图片。于是，就想如何免费管理图片，同时又能轻松共享他人。

在网上找了一些免费的随机图片 API，大部分处于不可用的状态，或者是需要注册登录，创建 API Token。

作为一名资深老年程序员，自然就想能通过编程实现。目标是从网络下载图片，

## 免费 CDN 加速

我的博客图片一直在用 GitHub 存储，通过 jsdelivr CDN 加速。于是就思考，如果能获取到 GitHub 存储的文件列表，就可以实现一个图片服务。

简单说下 jsdelivr CDN，它支持对 GitHub 中文件的加速访问。如位于我的仓库下的图片，通过对地址转为为 jsdelivr CDN 地址。

如下所示：

```bash
https://github.com/poloxue/public_images/default/0001.webp -> https://cdn.jsdelivr.net/gh/poloxue/public_images@latest/default/0001.webp
```

现在如果能顺利获取到仓库的图片文件列表，即可将 github 作为我们的图片图片存储，而无需花钱购买云存储实现。

如何获得 GitHub 文件列表呢？

## 查询 GitHub 图片列表

GitHub 支持接口获取仓库文件列表，如下所示，查询 user/repo 下某分支的情况。

```bash
https ://api.github.com/repos/{user}/{repo}/branches/{branch}。
```

JSON 返回体中，通过访问路径 `.commit.commit.tree.url` 拿到获取仓库文件列表的接口地址。其实主要是获取该分支最近的 commit hash。

演示案例，获取 `github.com/poloxue/public_images`

通过 `httpie` 执行请求，如下所示：

```bash
https ://api.github.com/repos/poloxue/public_images/branches/main
{
    // ...
    "commit": {
        "commit": {
            "tree": {
                "sha": "3859a482b15ed41bfb86ce073d6c500fef36910c",
                "url": "https://api.github.com/repos/poloxue/public_images/git/trees/3859a482b15ed41bfb86ce073d6c500fef36910c"
            }
        }
    }
}
```

通过 jq 解析请求结果，再次通过 httpie 请求，命令如下：

```bash
https $(https ://api.github.com/repos/poloxue/public_images/branches/main | jq -r '.commit.commit.tree.url+"?recursive=1"') | jq '.tree[].path'
```

如上的命令中通过 `?recursive=1` 实现遍历子目录，通过 '.tree[].path' 返回有文件目录。

返回结果如下：

```bash
.gitignore
README.md
beauties
beauties/0001.jpeg
beauties/0002.jpeg
beauties/0003.jpeg
beauties/0004.webp
beauties/0005.jpg
beauties/0006.webp
default
default/0001.webp
default/0002.webp
default/0003.webp
scenes
scenes/0001.webp
scenes/0002.webp
scenes/0003.webp
scenes/0004.webp
scenes/0005.webp
```


特别说明：接口的返回其实有数量限制，但这个限制并不是很大，个人使用无需担心。

## Web 服务

在了解如何使用GitHub 的接口后，我通过 aws 的 serverless 的能力，创建了一个简单的 Image Random API，将图片文件在仓库中的路径与 jsdelivr CDN 地址结合，随机返回一个图片地址。

接口定义：

- /image/random/{category}
- 输入参数：
  - category：str, 图片类型，即 github 仓库的子目名称；
- 返回结果：
  - image：str，图片地址，指定 category 类型下的一个图片地址；

核心的代码如下所示：

```python
import time
import random
import requests
from collections import defaultdict


class ImageService:
    def __init__(self):
        self._sha = None
        self._images = defaultdict(list)

        self._timeout = 60
        self._timestamp = 0

    def last_sha(self):
        last_timestamp = time.time()
        if last_timestamp - self._timestamp < self._timeout:
            return self._sha

        self._timestamp = last_timestamp
        data = requests.get(
            "https://api.github.com/repos/poloxue/public_images/branches/main"
        ).json()
        return data["commit"]["commit"]["tree"]["sha"]

    def get_images(self, category):
        last_sha = self.last_sha()
        if self._sha == last_sha:
            return self._images[category]

        self._images[category] = []
        self._sha = last_sha
        data = requests.get(
            f"https://api.github.com/repos/poloxue/public_images/git/trees/{last_sha}?recursive=1"
        ).json()

        for file in data["tree"]:
            fpath = file["path"]
            subdir = fpath.split("/")[0]
            if fpath.lower().endswith((".png", "jpg", "jpeg", "webp")):
                self._images[subdir].append(
                    f"https://cdn.jsdelivr.net/gh/poloxue/public_images@latest/{file['path']}"
                )
        return self._images[category]

    def random_image(self, category):
        images = self.get_images(category)
        if images:
            return random.choice(images)
```

如上方法，`random_image` 可提供给接口调用，从 GitHub 仓库返回一个随机图片。

请求示例，如下所示：

```bash
https ://api.poloxue.com/image/random/scenes | jq -r '.image'
```

输出结果：

```jsn
{
    "image": "https://cdn.jsdelivr.net/gh/poloxue/public_images@latest/scenes/0005.webp"
}
```

这只是服务的最小版本，还可以继续扩展，提供更多接口能力，如基于 Python 实现简单的裁剪缩放，皆是可行。

另外，这个 service 中还实现了简单的基于时间的缓存方案，另外当请求到分支最后的 hash 变化时才会更新 `self._images`。

唯一的遗憾就是，因为要提升共享能力，开发了一个简单的后端服务，没有免费云服务可用。

## 总结

本文介绍了如何基于 GitHub 实现一个简单的 random image API 服务，主要是了管理我的图片资源，同时实现了编程自由控制图片资源的的目标。

我计划将会利用这个的图片接口能力，自由更新我的桌面、iTerm 甚至是博客的背景图片，自己动手，丰衣足食。

