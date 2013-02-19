# SDK 使用文档编写规范

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


样例可以参考：<http://docs.qiniutek.com/v3/sdk/ruby/>


## SDK 文档书写细节（范例）

TODO
