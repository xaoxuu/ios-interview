# 第三方库

## 概念理解


### 常用的第三方库？



### SDWebImage 的实现原理？（加载图片过程）

1. 首先显示占位图
1. 在 webimagecache 中寻找图片对应的缓存，它是以 url 为数据索引先在内存中查找是否有缓存；
2. 如果没有缓存，就通过 md5 处理过的 key 来在磁盘中查找对应的数据，如果找到就会把磁盘中的数据加到内存中，并显示出来；
3. 如果内存和磁盘中都没有找到，就会向远程服务器发送请求，开始下载图片；
4. 下载完的图片加入缓存中，并写入到磁盘中；
5. 整个获取图片的过程是在子线程中进行，在主线程中显示。



### AFNetworking 底层原理分析

AFNetworking 是封装的 NSURLSession 的网络请求，由五个模块组成：

- NSURLSession：网络通信模块（核心模块） 对应 AFNetworking 中的 `AFURLSessionManager` 和对 HTTP 协议进行特化处理的 `AFHTTPSessionManager`
- Security：网络通讯安全策略模块  对应 `AFSecurityPolicy`
- Reachability：网络状态监听模块 对应 `AFNetworkReachabilityManager`
- Serialization：网络通信信息序列化、反序列化模块 对应 `AFURLResponseSerialization`
- UIKit：对于 iOS UIKit 的扩展库



### YYKit 的源码理解




### AsyncDisplayKit 是否会用




### 为什么用 AFNetworking




### 是否了解 ReactiveCocoa（RAC）框架？


> 详见这篇文章： [ReactiveCocoa 响应式编程初探](https://xaoxuu.com/blog/2016-07-20-reactive-cocoa/)
