---
title: "我用 GitHub 作为存储开发了一个随机图片 API"
date: 2023-11-17T15:35:36+08:00
draft: false
comment: true
description: "本文介绍如何基于 GitHub 为图片存储，通过 API 随机返回可用的图片地址。"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-11/2023-11-17-build-a-random-image-api-using-github-01.png)

本文介绍如何基于 GitHub 为图片存储，通过 API 随机返回可用的图片地址。

之所以研究它，主要是为了省钱，毕竟用啥 S3、七牛云、阿里云都是要花钱的。这套思路，gitee 应该也可以的，不过我看网上说，gitee 禁止图床开源啥的。而开发随机图片 API 只是为了验证是否能通过 GitHub 的 API 获取仓库中的文件，支持进一步开发其他管理工具。

## 前言

平时常用的桌面壁纸、终端背景图片，亦或是博客背景或文章封面，这些都离不开图片。于是，如何就想如何免费管理这些图片。

在网上找了一些免费的随机图片 API，大部分处于不可用的状态，或者是需要注册登录，创建 API Token。

作为一名老年程序员，自然就想能通过编程实现，实现图片自由。虽然也可以通过类似爬虫的思路实现，但还是希望都在自己的控制中，万一出现不好的图片就不好了。

## 免费 CDN 加速

我的博客图片一直在用 GitHub 存储，通过 jsdelivr CDN 加速。于是就思考，如果能获取到 GitHub 存储的文件列表，就可以实现一个图片服务。

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-11/2023-11-17-build-a-random-image-api-using-github-02.png)

简单说下 jsdelivr CDN，它支持对 GitHub 中文件的加速访问。如位于我的仓库下的图片，通过对地址转为为 jsdelivr CDN 地址。

如下的地址：

```bash
https://github.com/poloxue/public_images/default/0001.webp 
```

通过如下地址访问：

```bash
https://cdn.jsdelivr.net/gh/poloxue/public_images@latest/default/0001.webp
```

现在如果能顺利获取到仓库的图片文件列表，即可将 github 作为我们的图片图片存储，而无需花钱购买云存储实现。


## 查询 GitHub 图片列表

如何获得 GitHub 文件列表呢？这篇文章是讲如何将 GitHub 作为存储使用的，肯定要支持查询的，否则就没法玩了。

GitHub 支持接口获取仓库文件列表，基本流程是先通过某个接口查询仓库的信息，其中某个字段包含了最新的 hash，我们通过调用这个 hash，就能拿到这个 hash 下的文件列表。

仓库信息查看，即第一个接口，如下所示，如查询 user/repo 下某分支的情况。

```bash
curl https://api.github.com/repos/{user}/{repo}/branches/{branch}。
```

JSON 返回体中，通过访问路径 `.commit.commit.tree.url` 拿到获取仓库文件列表的接口地址。其实主要是获取该分支最近的 commit hash。

演示案例，获取 `github.com/poloxue/public_images`

通过 `httpie` 执行请求，如下所示：

```bash
$ curl https://api.github.com/repos/poloxue/public_images/branches/main
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
curl $(curl https://api.github.com/repos/poloxue/public_images/branches/main | jq -r '.commit.commit.tree.url+"?recursive=1"') | jq '.tree[].path'
```

如上的命令中通过 `?recursive=1` 实现遍历子目录，通过 '.tree[].path' 返回所有文件和目录。

返回结果如下：

```bash
.gitignore
README.md
beauties
beauties/0001.jpeg
beauties/0002.jpeg
default
default/0001.webp
default/0002.webp
scenes
scenes/0001.webp
scenes/0002.webp
```

特别说明：接口的返回其实有数量限制，但这个限制并不是很大，个人使用无需担心。如果想扩展扩展容量，可考虑创建多个分支，或者多个仓库啥的。

我的基本原则，把白嫖贯彻到底。

## 图片 API 服务

在了解如何使用GitHub 的接口后，我通过 gin 开发一个随机 API，创建了一个简单的 Random Image API。实现方案也不复杂，通过 GitHub 接口获取图片列表，随机选择其中一张图片，把它在仓库中的路径与 jsdelivr CDN 地址拼接，随机返回一个图片地址。


接口定义：

- Path: /image/random/{category}
- Parameters：
  - category：str, 图片类型，即上面 github 仓库的子目名称；
- Return：
  - image：str，图片地址，指定 category 类型下的一个图片地址；

接下来，我就来尝试实现这个服务。

### main 函数

我直接使用 gin 框架开发这个 demo API。`main` 入口代码如下所示：

```go
var imageContainer imageContainer

func init() {
  imageContainer = NewImageContainer()
}

func GetRandomImage(ctx *gin.Context) {
  category := c.Param("category")
  imageURL, err := imageContainer.RandomImage(category)
  // ...
  c.JSON(http.StatusOK, gin.H{"image": imageURL})
}

func main() {
  router := gin.Default()
  router.GET("/image/random/:category", GetRandomImage)
  router.Run(":8080")
}
```

`gin` 创建和路由的部分没啥可介绍的。重点是，我创建了一个 `ImageContainer` 用于连接 GitHub 随机拿到随机的图片地址。

为什么我把 `imageContainer` 声明为全局变量，主要想着这样设计个本地缓存更容易些，不然一直请求 GitHub API 可不是个好事情，容易触发 GitHub 请求p频率限制。

## ImageContainer

首先，定义下 `ImageContainer` 类型和它的创建函数，代码。

如下所示：

```go
type ImageContainer struct {
  repo   string
  branch string
  images map[string][]string
}

func NewImageContainer(repo, branch string) *ImageContainer {
  return &ImageContainer{
    repo:   repo,
    branch: branch,
  }
}
```

构造函数 `NewImageContainer` 创建时，要指定仓库 `repo` 和分支 `branch`。`ImageContainer` 中的 `images` 字段用于按分类保存图片列表。对于 API 场景，使用 `sync.Map` 是个更好的选择，因为会存在并发竞争的问题。

随机图片的获取流程可描述为三个步骤，分别是查询最新 commit hash、基于 commit hash 获取 trees 图片列表和从图片列表中随机拿到一张图片返回。

随机图片 API 的核心部分代码，如下所示：

```go
func (s *ImageContainer) RandomImage() {
	lastHash, err := s.LastHash()
	if err != nil {
		return "", err
	}

	images, err := s.QueryImages(category, lastHash)
	if err != nil {
		return "", err
	}

	if len(images) == 0 {
		return "", errors.New("No image found")
	}
	return images[rand.Intn(len(images))], nil
}
```

其他部分如 `LastHash`、`QueryImages` 函数代码比较啰嗦，就不介绍了。不喜欢把太丑的代码贴在文章里。有兴趣可查看源码仓库 [poloxue/random_image_from_github](https://github.com/poloxue/random_image_from_github)。


## 测试效果

请求示例，如下所示：

```bash
curl http://localhost:8080/image/random/scenes
```

输出结果：

```jsn
{
    "image": "https://cdn.jsdelivr.net/gh/poloxue/public_images@latest/scenes/0005.webp"
}
```

这只是服务的最小版本，还可以继续扩展，提供更多接口能力，甚至可以上传下载图片，还有生成缩略图等等。本案例只是测试代码，很多场景是没有实现的。

## 总结

本文介绍了如何基于 GitHub 实现一个简单的 random image API 服务，主要还是展示将 GitHub 作为替代 S3 存储服务使用。对于个人而言，肯定是能白嫖，就不要花钱，哈哈。

因为 GitHub 其实是提供了完整一套的增删改查接口的，基于它，我可以开发一个图床管理 app。还有，进一步，还可以基于它开发如 neovim、vscode 的博客图片插件，提升 markdown 写作效率。

我的博客地址：[以 GitHub 作为图片存储创建随机图片 Service API](https://www.poloxue.com/posts/2023-11-17-build-a-random-image-api-using-github/)
