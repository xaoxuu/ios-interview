# 数据持久化（数据库）

## 概念理解

### iOS 数据持久化有哪几种？

NSUserDefaults、归档、沙盒文件、数据库、CoreData、KeyChain(钥匙串) 等多种方式。其中 KeyChain 是保存到沙盒范围以外的地方，也就是与沙盒无关。



## 数据库

### FMDB 数据结构变化升级

1. 使用 `columnExists:inTableWithName` 方法判断数据表中是否存在字段

2. 如果不存在，则添加，如：向 bbb 表中添加 aaa 字段
  ```sql
  ALTER TABLE bbb ADD 'aaa' TEXT
  ```


### 数据库如何解决资源竞争问题？
