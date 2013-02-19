# SDK单元测试规范

## 单元测试写法

一般来讲，单元测试的代码量是业务逻辑的3倍，针对每一个业务逻辑要写3个测试：YES逻辑断言, FALSE逻辑断言, 异常出错逻辑断言。

单元测试需覆盖所有的业务逻辑。

单元测试代码范式如下：

```

// 初始化测试环境

before :all do

    // 使用测试密钥（AccessKey/SecretKey）接入
    establish_connection
    
    // 创建测试Bucket
    mk_unique_test_bucket 
    
end


// 测试指定功能点

context :some_feature do

    // YES逻辑断言
    it "should be YES" do
      // do some thing
    end

    // FALSE逻辑断言
    it "should be FALSE" do
      // do some thing
    end

    // 异常出错逻辑断言
    it "should be Raise Err" do
      // do some thing
    end
end

context :yet_another_feature do
   // ...
end


// 清场

after :all do
    drop_unique_test_bucket
    clean_tmpfile
end
```

## 自动化测试

使用 Travis-CI 作为 SDK 的自动化测试工具。

在 `<SDK>.git` 代码库根目录下添加 `.travis.yml` 文件，接入参考：<http://about.travis-ci.org/docs/>

缺省每个 `<SDK>.git` 代码库会接入 Travis-CI, 开发人员可以根据实际需要更改 `.travis.yml` 文件里边的配置项。

自动化测试流程参考如下图示：

![自动化测试流程图](http://xudaoli.qiniudn.com/talks/teamwork-with-git-flow/assets/github-pull-request-with-travis-ci.png)

接入 Travis-CI 后，开发者向 `<SDK>.git` 代码主库提交 Pull Request 时，会触发 Travis-CI 自动跑 SDK 单元测试并汇报测试结果。如下示例：

![自动化测试流程图1](http://xudaoli.qiniudn.com/talks/teamwork-with-git-flow/assets/github-pull-request-with-travis-ci-step-1.png)