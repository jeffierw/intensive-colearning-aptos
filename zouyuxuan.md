---
timezone: Asia/Shanghai
---


# {zouyuxuan}

1. 我叫邹宇轩，5年开发经验，掌握go，rust，cairo，move开发语言
2. 不逼自己一把怎么能坚持下来

## Notes

<!-- Content_START -->

### 2024.09.07
#### 1、阅读aptos官方文档，了解object使用方法，阅读aptos源代码，了解object在智能合约中如何使用
#### 2、编译aptos智能合约，部署测试上网络

笔记内容

### 2024.09.08
#### aptos client 使用
#### 1、创建账户 aptos init --profile 
```
# 创建一个tz账户
aptos init --profile tz 
aptos 会在当前文件夹目录下创建一个.aptos隐藏文件夹，文件内容包括账户名称，公钥，私钥等信息

# 领水 address 制定为自己的账户
1、aptos account fund-with-faucet --profile tz 
2、curl -X POST
'https://faucet.devnet.aptoslabs.com/mint?amount=10000&address=0xd0f523c9e73e6f3d68c16ae883a9febc616e484c4998a72d8899a1009e5a89d6'

# 查看tz账户余额，使用--url参数指定环境
aptos account balance --account tz --url https://fullnode.devnet.aptoslabs.com

# 使用tz账户给0x9002a65796acd991b7f0bca4bc7e6428821fa393ba95ba356cf7769435d12250 转账
aptos account transfer --account 0x9002a65796acd991b7f0bca4bc7e6428821fa393ba95ba356cf7769435d12250 --amount 1000 --profile tz

```

### 2024.09.09
#### 熟悉move语言基本语法，编写基础智能合约
### 2024.09.10
#### 创建move合约，编译部署
```
1、初始化一个名为game的智能合约项目文件
aptos move init --name game
2、编译智能合约
aptos move compile
```
`注意事项：` 如果编译过程中may take a little while to download git dependencies...下载依赖报错
替换 [dependencies.AptosFramework]中git地址为 "https://gitee.com/WGB5445/aptos-core.git"
### 2024.09.11
#### 阅读aptos move example源码
### 2024.09.12
#### move 语言函数修饰符
核心概念
函数修饰符是用来赋予函数特殊能力的一组关键字。主要有以下几类

- 可见性
  - 无Public，私有函数，仅限 module 内部调用
  - friend public，模块内部函数，同包模块之间可以调用
  - Public，模块公开函数，所有模块都可以调用
- 全局存储引用
  - acquires，当需要使用 move_from，borrow_global，borrow_global_mut 访问地址下的资源的时候，需要使用 acquires 修饰符
- 链下
  - entry，修饰后，该方法可由链下脚本调用
### 2024.09.13

### 2024.09.14
认识 OBJECT
什么是 Aptos object？
1. 对象是单个地址的资源容器，用于储存资源
2. 对象提供了一种集中式资源控制与所有权管理的方法

创建并转移对象实例
我们可以通过 aptos_framework 库下的 object 来实现对象的功能。
```rust
module my_addr::object_playground {
  use std::signer;
  use aptos_framework::object::{self, ObjectCore};

  entry fun create_and_transfer(caller: &signer, destination: address) {
    // Create object
    let caller_address = signer::address_of(caller);
    let constructor_ref = object::create_object(caller_address);

    // Set up the object

    // Transfer to destination
    let object = object::object_from_constructor_ref<ObjectCore>(&constructor_ref);
    object::transfer(caller, object, destination);
  }
}
```

两类对象
可删除普通对象
不可删除对象

- 命名对象，通过固定的 signer 和特定的 seed 生成唯一地址的对象，1个地址只能生成1个
- 粘性对象，通过 signer 生成的对象，1个地址可以生成多个
  
### 2024.09.15
OBJECT 配置与使用
object 有哪些配置？

1. 扩展对象：将对象变成可动态配置的，可以往里面添置新的 Struct 资源
2. 转移管理：可以开启或者禁用 object transfer 功能
3. 受控转移：仅可使用一次转移功能
4. 允许删除：允许删除对象
### 2024.09.16
学习aptos framework中关于事件，时间，transfer方法的使用

### 2024.09.17
Aptos 可替代资产标准（Fungible Asset，简称 FA）为 Move 生态系统提供了定义可替代资产的标准和类型安全的方式，取代了以前的 coin 模块。FA 标准允许无缝铸造、转移和自定义各种用途的可替代资产，如货币和真实世界资产（RWA）。它具有更强的自定义能力，并能确保去中心化应用（dApp）一致识别和处理这些资产。

**FA 标准通过两个 Move 对象实现：**
1. **Object<Metadata>**：表示资产的元数据，如名称、符号和小数点位数。
2. **Object<FungibleStore>**：存储账户所拥有的资产数量。

**FA 标准的优势：**
- 与 coin 模块相比，FA 更具自定义性，支持通过智能合约扩展。
- 自动跟踪账户资产余额，无需额外注册资源。

**创建 FA 资产步骤：**
1. 创建不可删除的 Object 持有元数据。
2. 生成权限 Refs，如铸币权限（MintRef）、转移权限（TransferRef）和销毁权限（BurnRef）。
3. 铸造和转移资产。

**事件与操作：**
- **提现**：从主存储取出资产。
- **存款**：向主存储存入资产。
- **转移**：将资产从一个账户转移到另一个账户。
- **余额查询**：检查账户的资产余额。
- **冻结状态查询**：检查账户是否被冻结。

**高级功能：可调度资产**

FA 还支持自定义存取钩子函数，允许资产发行者实现自定义的存取逻辑。

**迁移**

FA 标准兼容 coin 模块，旧的 coin 模块将自动迁移至 FA，并为其创建相应的元数据。
### 2024.09.18
#### event 事件
在有drop和store的属性的结构体上面添加#[event]宏
```
#[event]
struct TransferEvent has drop, store {
    sender: address,
    receiver: address,
    amount: u64
}
```
使用事件
```
# 导入事件
use aptos_framework::event;

# 调用事件
event::emit(TransferEvent { sender: address,
    receiver: address,
    amount: 1}
    );
```

### 2024.09.19
学习aptos-move例子
### 2024.09.20
学习aptos-move 中bonding_curve_launchpad例子
合约初始化函数使用
```
init_module函数必须是私有的，不能返回任何参数

 fun init_module(account: &signer) {
        let signer_extender = object::generate_extend_ref(
            &object::create_sticky_object(@bonding_curve_launchpad)
        );
        move_to(account, Pairs { signer_extender });
    }
```
### 2024.09.21
1. 对象是单个地址的资源容器，用于储存资源
2. 对象提供了一种集中式资源控制与所有权管理的方法

创建并转移对象实例
我们可以通过 aptos_framework 库下的 object 来实现对象的功能。
```rust
module my_addr::object_playground {
  use std::signer;
  use aptos_framework::object::{self, ObjectCore};

  entry fun create_and_transfer(caller: &signer, destination: address) {
    // Create object
    let caller_address = signer::address_of(caller);
    let constructor_ref = object::create_object(caller_address);

    // Set up the object

    // Transfer to destination
    let object = object::object_from_constructor_ref<ObjectCore>(&constructor_ref);
    object::transfer(caller, object, destination);
  }
}
```

两类对象
可删除普通对象
不可删除对象

- 命名对象，通过固定的 signer 和特定的 seed 生成唯一地址的对象，1个地址只能生成1个
- 粘性对象，通过 signer 生成的对象，1个地址可以生成多个
### 2024.09.22
函数
 - Public functions
   ```
   public fun get_aptogotchi(owner_addr: address) { }
   ```
    - View Functions
   - ```
     #[view]
     public fun get_name(user_addr: address): String acquires AptoGotchi { }
     ```
    - Inline Functions
     ```
     inline fun get_aptogotchi_internal(creator_addr: &address) { }
     ```
- Private Functions私有函数是合约中没有public修饰符的函数。其他模块（合约）无法访问它。
  ```
  fun get_aptogotchi_address(creator_addr: &address): (address) { }
  ```

### 2024.09.23
学习aptos-move Dao例子 

### 2024.09.24
构建参赛项目
### 2024.09.25
学习水龙头程序，根据自己的项目编写新的水龙头程序
### 2024.09.26
编译部署程序，优化代码
### 2024.09.27
``` 
// 关键结构体
struct SimpleMap<Key, Value> has copy, drop, store

length()  // 获取长度
new()  //创建空的SimpleMap
new_from()  //使用vector类型的keys和values创建Simple
borrow()
borrow_mut()
contains_key()  // 是否含有某个key
destory_empty()
add()     //添加key/value，key不能已经存在
add_all()  // 联想new_from()
upsert()   // 插入或更新
keys()    // 返回vector<key>
values()   // 返回vector<value>
to_vec_pair() // 返回(vector<Key>, vector<Value>)
destory()   // ???
remove()   // key必须存在
find()     
```
<!-- Content_END -->