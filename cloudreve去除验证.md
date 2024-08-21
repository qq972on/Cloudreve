<iframe height="1" width="1" style="top: 0px; l

#### 破解 Cloudreve V3 Pro，解锁所有功能，附带教程和源码下载 

## 1. 原因

以下是来自于部分网友的观点，不代表本人。

> 还是因为Cloudreve只顾Pro版圈钱，不管issues提议，一堆Bug不修。论坛提出来你当没看见，修好了发Github你举报，用户的心都被你们给伤透了。所以正值v4即将发布之际，公布v3完整版"解锁"方式。

![img](https://cloudreve.org/imgs/feature1.png)

## 2. 后端

### 2.1 分析

众所周知，捐助版会检测授权文件 key.bin，没有它是连程序都打不开的。那有人说了，在 app.go 的 InitApplication 函数里删掉就可以了？

开发者能让你这么简单就破开吗，试过之后发现还是打不开程序。

他说的对，但不完全对，猫腻就藏在程序的依赖库里，仔细看这个库 [https://github.com/abslant/gzip/blob/v0.0.9/handler.go#L60](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fabslant_2Fgzip_2Fblob_2Fv0_2e0_2e9_2Fhandler_2ego_23L60)

看似只是一个fork版，但会在前端 main.xxx.chunk.js 中插入跳转官网403的代码，作者的用户名为 abslant，乍一看不认识。

打开这个博客 [https://hfo4.github.io/](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fhfo4_2egithub_2eio_2F) ，注意头像下的联系邮箱，发现这就是开发者 Aaron 的小名。

这一切就说得通了，都是作者搞的鬼。看过社区版源码的都知道，没看过的等你尝试用git对比整个仓库的时候就知道了。

### 2.2 改动

1、首先将被加料的依赖项替换为原版

```sh
github.com/abslant/mime => github.com/HFO4/aliyun-oss-go-sdk
github.com/abslant/gzip => github.com/gin-contrib/gzip
```

（VSC编辑器全局搜索，直接替换）

2、bootstrap/app.go 不用多说，那个读取 `[]byte{107, 101, 121, 46, 98, 105, 110}` 的就是授权文件

3、routers/router.go 第128行 `r.Use(gzip.GzipHandler())` 改为 `r.Use(gzip.Gzip(gzip.DefaultCompression, gzip.WithExcludedPaths([]string{"/api/"})))`

（如果改完还是自动引入就把 go.sum 删了）

4、然后是一些小变动：

pkg/hashid/hash.go 最后一个函数 `constant.HashIDTable[t]` 改为 `t` 基本上到这里就完成了。

注意：前端打包时要保持目录结构 `assets.zip/assets/build/{前端文件}`

## 3. 前端

### 3.1 插曲

忙活了半天，终于把程序跑起来了，打开页面一看，好家伙 Backend not running 还是进不去，怎么想都进不去，因为前端还有一层验证。

### 3.2 改动

但注意 "任何前端加密和混淆都是纸老虎，自己玩玩无所谓，重要业务千万别乱来" 前端验证很好破解，还是先检查依赖项。

1、打开 package.json 头两行就是这个万恶的 abslant，删掉 `"@abslant/cd-image-loader"` 和 `"@abslant/cd-js-injector"`。

2、把引用它们的地方删掉就行...了吗 ?

位置在 config/webpack.config.js:35_625 和 src/component/FileManager/FileManager.js:16_109

之后进是能进网盘了，但你想测试上传一个文件的时候就傻眼了，明明什么也没动，就是传不上去

报错 Cannot read properties of null (reading 'code')，那是继3.5.3之后新增的一处验证 将 src/component/Uploader/core/utils/request.ts 第 12 行整个 const 替换为以下内容即可解决

```js
const baseConfig = {
    transformResponse: [
        (response: any) => {
            try {
                return JSON.parse(response);
            } catch (e) {
                throw new TransformResponseError(response, e);
            }
        },
    ],
};
```

## 其它

除了去除验证，Plus版本还增加了几处功能优化，修复遗留Bug，感兴趣的可以下载体验一下。

但因为是3.8.3泄露版和主线版拼凑而来的，存在不稳定因素，建议不要用于生产环境 如果怕我在里面加料，可以自行检查源码，这程序十分的珍贵，尽快下载存档！

主地址：

- [cloudreveplus-windows-amd64v2.zip](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fcloudreve_2FCloudreve_2Ffiles_2F14327258_2Fcloudreveplus_2dwindows_2damd64v2_2ezip)
- [cloudreveplus-linux-amd64v2.zip](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fcloudreve_2FCloudreve_2Ffiles_2F14327249_2Fcloudreveplus_2dlinux_2damd64v2_2ezip)
- [cloudreveplus-linux-arm7.zip](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fcloudreve_2FCloudreve_2Ffiles_2F14327254_2Fcloudreveplus_2dlinux_2darm7_2ezip)
- [cloudreveplus-source-nogit.zip](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fcloudreve_2FCloudreve_2Ffiles_2F14327256_2Fcloudreveplus_2dsource_2dnogit_2ezip)

备用地址（以图片方式上传可以分到aws的地址，比githubusercontent快一些，但要分卷手动改名）:

- [cloudreveplus-source-nogit.zip](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fcloudreve_2Ffrontend_2Fassets_2F100983035_2F4fe3ae36_2d275d_2d41e9_2d89fe_2d2a746f512bde)
- [cloudreveplus-linux-amd64v2.001](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fcloudreve_2Ffrontend_2Fassets_2F100983035_2F71dab1b8_2d8a01_2d4609_2dbf1d_2dab8f6c5df57d) | [cloudreveplus-linux-amd64v2.002](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fcloudreve_2Ffrontend_2Fassets_2F100983035_2F423cb9cb_2d9dae_2d47e9_2dbaf3_2d43a48202fe06)
- [cloudreveplus-linux-arm7.001](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fcloudreve_2Ffrontend_2Fassets_2F100983035_2Fa03f6c72_2d3ee8_2d44f4_2d96ed_2dca385bc87c5c) | [cloudreveplus-linux-arm7.002](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fcloudreve_2Ffrontend_2Fassets_2F100983035_2Fe3f9a73d_2d9019_2d4c60_2da41b_2db53a9184aad9)
- [cloudreveplus-windows-amd64v2.001](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fcloudreve_2Ffrontend_2Fassets_2F100983035_2Fa6d68487_2d3f40_2d4f6c_2d9cab_2d857d4128fb7d) | [cloudreveplus-windows-amd64v2.002](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fgithub_2ecom_2Fcloudreve_2Ffrontend_2Fassets_2F100983035_2Fc3620b29_2d8ced_2d4aa7_2da02b_2d8d14c0bf4815)

备用地址 2：

本帖有隐藏内容，请您[回复](https://bbs.yiove.com/post-create-75101.htm)后查看。



参考资料：[https://hostloc.com/thread-1276280-1-1.html](https://bbs.yiove.com/gowild.htm?url=https_3A_2F_2Fhostloc_2ecom_2Fthread_2d1276280_2d1_2d1_2ehtml)

eft: 0px; border: medium; visibility: hidden;"></iframe>