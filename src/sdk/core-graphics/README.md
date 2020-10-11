# 图形和动画

## 概念理解

### 图像的压缩方式

压缩图片质量

一般情况下使用UIImageJPEGRepresentation或UIImagePNGRepresentation方法实现。

压缩图片尺寸

一般通过指定压缩的大小对图像进行重绘



### 如何计算图片加载内存中所占的大小

图片内存大小的计算公式 宽度 * 高度 * bytesPerPixel/8。

bytesPerPixel : 每个像素所占的字节数。

RGBA颜色空间下 每个颜色分量由32位组成

所以一般图片的计算公式是 wxhx4



### UIView 动画与核心动画的区别?

核心动画只作用在layer.
核心动画修改的值都是假像.它的真实位置没有发生变化.
当需要与用户进行交互时用UIView动画,不需要与用户进行交互时两个都可以.



### 当我们要做一些基于 CALayer 的动画时，有时需要设置 layer 的锚点来配合动画，这时候我们需要注意什么？

需要注意的是设置锚点会引起原来 position 的变化，可能会发生不符合预期的行为，所以要做一下转化，示例代码如下：
```objc
// 为 layer 的动画设置不同的 anchor point，但是又不想改变 view 原来的 position，则需要做一些转换。
- (void)setAnchorPoint:(CGPoint)anchorPoint forView:(UIView *)view {
    // 分别计算原来锚点和将更新的锚点对应的坐标点，这些坐标点是相对该 view 内部坐标系的。
    CGPoint oldPoint = CGPointMake(view.bounds.size.width * view.layer.anchorPoint.x,
                                   view.bounds.size.height * view.layer.anchorPoint.y);
    CGPoint newPoint = CGPointMake(view.bounds.size.width * anchorPoint.x,
                                   view.bounds.size.height * anchorPoint.y);

    // 如果当前 view 有做过 transform，这里要同步计算。
    oldPoint = CGPointApplyAffineTransform(oldPoint, view.transform);
    newPoint = CGPointApplyAffineTransform(newPoint, view.transform);

    // position 是当前 view 的 anchor point 在其父 view 的位置。
    CGPoint position = view.layer.position;
    // anchor point 的改变会造成 position 的改变，从而影响 view 在其父 view 的位置，这里把这个位移给计算回来。
    position.x = position.x + newPoint.x - oldPoint.x;
    position.y = position.y + newPoint.y - oldPoint.y;

    view.translatesAutoresizingMaskIntoConstraints = YES;
    view.layer.anchorPoint = anchorPoint; // 设置了新的 anchor point 会改变位置。
    view.layer.position = position; // 通过在 position 上做逆向偏移，把位置给移回来。
}
```

<!-- ** -->



### 贝塞尔曲线的绘制（折线图）





### 什么时候会触发离屏渲染




