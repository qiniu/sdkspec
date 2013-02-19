# SDK 使用文档编写规范

SDK 使用文档位于代码库的 `docs/README.md` ，文档需基于 [Markdown](http://markdown.tw/) 语法格式编写。

开发者可以基于 Github 提供的在线编辑器编写，也可以使用如下 Markdown 编辑器。

## Markdown Editor

Mac

* [Mou](http://mouapp.com/)

Linux

* [ReText](http://sourceforge.net/p/retext/home/ReText/)

Windows

* [MarkdownPad](http://markdownpad.com/)
* [MarkPad](http://code52.org/DownmarkerWPF/)

Online Tools

* [Markable.in](http://markable.in/)
* [Dillinger.io](http://dillinger.io/)

Chrome

* [MaDe](https://chrome.google.com/webstore/detail/oknndfeeopgpibecfjljjfanledpbkog) (Chrome)

Others

* [Sublime Text 2](http://www.sublimetext.com/2) + [MarkdownEditing](http://ttscoff.github.com/MarkdownEditing/)


## SDK 文档书写目录结构（范式）

- SDK 简介说明

- SDK 接入/安装/环境依赖说明

- SDK 使用篇

    - 文件上传
        - 生成上传授权凭证（uploadToken）
        - 上传文件
        - 断点续上传（如果SDK支持）
    
    - 文件下载
        - 公有资源下载（[引导至API说明](http://docs.qiniutek.com/v3/api/io/#public-download)）
        - 私有资源下载
            - 生成下载授权凭证（downloadToken）
        - 高级特性
            - 断点续下载（[引导至API说明](http://docs.qiniutek.com/v3/api/io/#download-by-range-bytes)）
            - 自定义 404 NotFound（[引导至API说明](http://docs.qiniutek.com/v3/api/io/#download-if-notfound)）
    
    - 文件操作
        - 查看文件基本属性信息
        - 复制文件
        - 移动文件
        - 删除文件
        - 批量操作（查看、复制、移动、删除）
    
    - 云处理
        - 图像
            - 查看图片属性信息
            - 查看图片EXIF信息
            - 图像在线处理（缩略、裁剪、旋转、转化）
            - 图像在线处理（缩略、裁剪、旋转、转化）后并持久化存储
        - 音频
        - 视频

## 样例参考

SDK 文档源文件参考：<https://github.com/qiniu/ruby-sdk/blob/develop/docs/README.md>

Markdown 源文件参考: <https://raw.github.com/qiniu/ruby-sdk/develop/docs/README.md>