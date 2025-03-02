---
title: "Part 2. 创建一个 Thrift Server"
linkTitle: "创建一个 Thrift Server"
weight: 2
description: >

---

Volo-Thrift 是一个 RPC 框架，既然是 RPC，底层就需要两大功能：
1. Serialization 序列化
2. Transport 传输

IDL 全称是 `Interface Definition Language`，接口定义语言。

## Why IDL

如果我们要进行 RPC，就需要知道对方的接口是什么，需要传什么参数，同时也需要知道返回值是什么样的，就好比两个人之间交流，需要保证在说的是同一个语言、同一件事。
这时候，就需要通过 IDL 来约定双方的协议，就像在写代码的时候需要调用某个函数，我们需要知道函数签名一样。

Thrift IDL 是一套跨语言的全栈式 RPC 解决方案，具体的语法可以看参考 [thrift-missing-guide](https://diwakergupta.github.io/thrift-missing-guide/) 或官方 [Thrift interface description language](http://thrift.apache.org/docs/idl)。

## 编写 IDL

为了创建一个 Thrift 项目，我们需要先编写一个 Thrift IDL。

在你的工作目录下，我们先执行以下命令：

```bash
$ mkdir volo-example
$ cd volo-example
$ mkdir idl
$ vim idl/volo_example.thrift
```

随后，我们输入以下内容：

```thrift
namespace rs volo.example

struct Item {
    1: required i64 id,
    2: required string title,
    3: required string content,

    10: optional map<string, string> extra,
}

struct GetItemRequest {
    1: required i64 id,
}

struct GetItemResponse {
    1: required Item item,
}

service ItemService {
    GetItemResponse GetItem (1: GetItemRequest req),
}
```

保存退出后，我们执行以下命令：

```bash
$ volo init volo-example idl/volo_example.thrift
```

**这里我们使用`init`命令，后面跟了我们项目的名字，意思是需要生成模板代码。在末尾，需要指定一个 IDL 表示 server 使用的 IDL。**

如果只需要增加一个 IDL（如 client 的 IDL）而不需要生成模板的话，如：

```bash
$ volo idl add idl/volo_example.thrift
```

| 插播一个广告，volo 工具还支持从 git 下载 IDL 并生成代码哦，如：

```bash
$ volo idl add -g git@github.com:org/repo.git -r main /path/to/your/idl.thrift
```

| 感兴趣可以直接输入 volo 看详细用法~ 接下来回到正题~

这时候，我们整个目录的结构如下：

```bash
.
├── Cargo.toml
├── idl
│   └── volo_example.thrift
├── rust-toolchain.toml
├── src
│   ├── bin
│   │   └── server.rs
│   └── lib.rs
└── volo-gen
    ├── Cargo.toml
    ├── build.rs
    ├── src
    │   └── lib.rs
    └── volo.yml
```

然后，我们打开 `src/lib.rs`，在 impl 块中加入方法的实现，最终的代码应该是这样的：

```rust
#![feature(type_alias_impl_trait)]

pub struct S;

#[volo::async_trait]
impl volo_gen::volo::example::ItemService for S {
    // 这部分是我们需要增加的代码
    async fn get_item(
        &self,
        _req: volo_gen::volo::example::GetItemRequest,
    ) -> core::result::Result<volo_gen::volo::example::GetItemResponse, volo_thrift::AnyhowError>
    {
        Ok(Default::default())
    }
}
```

然后执行：

```bash
$ cargo update
$ cargo build
```

这时候，就会发现 [OUT_DIR 目录](https://doc.rust-lang.org/cargo/reference/environment-variables.html#environment-variables-cargo-sets-for-build-scripts)下多出来一个 volo_gen.rs 的文件了。

然后执行以下命令，即可把我们的 server 端跑起来：

```bash
$ cargo run --bin server
```

至此，我们已经能把我们的 server 跑起来啦！
