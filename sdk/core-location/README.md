# 定位与地图

## 基本流程

### CoreLocation 模块与常见类？

- `CLLocation`：表示某个位置的地理信息，比如经纬度、海拔等
- `CLLocationManager`：定位管理器，可以理解为定位不能自己工作，需要有个管理者对它进行全过程监督。
- `CLGeocoder`：地理编码，分为两种
  - 正向地理编码：根据位置信息，获取具体的经纬度等信息
  - 反向地理编码：根据给定的经纬度等信息，获取位置信息
- `CLPlacemark`：位置信息，包含的信息如国家、城市、街道等
- `CLLocationManagerDelegate`：定位代理，不管是定位成功与失败，都会有相应的代理方法回调



### CoreLocation 定位的流程？

1. CLLocationManager 发起定位，定位成功或者失败都会回调 CLLocationManagerDelegate 中相应的代理方法
2. 在成功的代理方法中获取 CLLocation 对象，进而获取经纬度
3. 通过 CLGeocoder 获取经纬度对应的位置信息 CLPlacemark
4. 通过 CLPlacemark 获取具体的位置信息



## 应用能力

### 如何解决火星坐标偏移问题？




<br>

> **参考资料**
> - [iOS开发之定位](https://www.jianshu.com/p/022d7f58f9db)
