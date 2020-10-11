# 性能优化

## 概念理解

### 造成 tableView 卡顿的原因有哪些，如何避免？

1. 最常用的就是 cell 的重用， 注册重用标识符

  如果不重用 cell，每当一个 cell 显示到屏幕上时，就会重新创建一个新的 cell。如果有很多数据的时候，就会堆积很多 cell。
  如果重用 cell，为 cell 创建一个 ID，每当需要显示 cell 的时候，都会先去缓冲池中寻找可循环利用的 cell，如果没有再重新创建 cell。

2. 避免 cell 的重新布局

  cell 的布局填充等操作比较耗时，一般创建时就布局好，如可以将 cell 单独放到一个自定义类，初始化时就布局好。

3. 提前计算并缓存 cell 的属性及内容

  当我们创建 cell 的数据源方法时，编译器并不是先创建 cell 再定 cell 的高度，而是先根据内容一次确定每一个 cell 的高度，高度确定后，再创建要显示的 cell，滚动时，每当 cell 进入屏幕都会计算高度，提前估算高度告诉编译器，编译器知道高度后，紧接着就会创建 cell，这时再调用高度的具体计算方法，这样可以方式浪费时间去计算显示以外的 cell

4. 减少 cell 中控件的数量

  尽量使 cell 得布局大致相同，不同风格的 cell 可以使用不用的重用标识符，初始化时添加控件。

5. 不要使用 ClearColor，无背景色，透明度也不要设置为0，渲染耗时比较长。

6. 使用局部更新

如果只是更新某组的话，使用 `reloadSection` 进行局部更新。

7. 加载网络数据，下载图片，使用异步加载，并缓存

8. 少使用addView 给cell动态添加view

9. 按需加载 cell，cell 滚动很快时，只加载范围内的 cell

10. 不要实现无用的代理方法

11. 缓存行高：`estimatedHeightForRow` 不能和 `HeightForRow` 里面的 `layoutIfNeed` 同时存在，这两者同时存在才会出现“窜动”的 bug。所以我的建议是：只要是固定行高就写预估行高来减少行高调用次数提升性能。如果是动态行高就不要写预估方法了，用一个行高的缓存字典来减少代码的调用次数即可。

12. 不要做多余的绘制工作。在实现 `drawRect:` 的时候，它的 rect 参数就是需要绘制的区域，这个区域之外的不需要进行绘制。例如上例中，就可以用 CGRectIntersectsRect、CGRectIntersection或CGRectContainsRect 判断是否需要绘制 image 和 text，然后再调用绘制方法。

13. 预渲染图像。当新的图像出现时，仍然会有短暂的停顿现象。解决的办法就是在 bitmap context 里先将其画一遍，导出成 UIImage 对象，然后再绘制到屏幕。

14. 使用正确的数据结构来存储数据。



### 如何提升 tableview 的流畅度

本质上是降低 CPU、GPU 的工作，从这两个大的方面去提升性能。

CPU：对象的创建和销毁、对象属性的调整、布局计算、文本的计算和排版、图片的格式转换和解码、图像的绘制

GPU：纹理的渲染

#### 卡顿优化在 CPU 层面


尽量用轻量级的对象，比如用不到事件处理的地方，可以考虑使用 CALayer 取代 UIView

不要频繁地调用 UIView 的相关属性，比如 frame、bounds、transform 等属性，尽量减少不必要的修改

尽量提前计算好布局，在有需要时一次性调整对应的属性，不要多次修改属性

Autolayout 会比直接设置 frame 消耗更多的 CPU 资源

图片的 size 最好刚好跟 UIImageView 的 size 保持一致

控制一下线程的最大并发数量

尽量把耗时的操作放到子线程

文本处理（尺寸计算、绘制）

图片处理（解码、绘制）

#### 卡顿优化在 GPU层面


尽量避免短时间内大量图片的显示，尽可能将多张图片合成一张进行显示

GPU能处理的最大纹理尺寸是 4096x4096，一旦超过这个尺寸，就会占用 CPU 资源进行处理，所以纹理尽量不要超过这个尺寸

尽量减少视图数量和层次

减少透明的视图（alpha<1），不透明的就设置 opaque 为 YES

尽量避免出现离屏渲染

#### iOS 保持界面流畅的技巧

1. 预排版，提前计算

  在接收到服务端返回的数据后，尽量将 CoreText 排版的结果、单个控件的高度、cell 整体的高度提前计算好，将其存储在模型的属性中。需要使用时，直接从模型中往外取，避免了计算的过程。

  尽量少用 UILabel，可以使用 CALayer 。避免使用 AutoLayout 的自动布局技术，采取纯代码的方式

2. 预渲染，提前绘制

  例如圆形的图标可以提前在，在接收到网络返回数据时，在后台线程进行处理，直接存储在模型数据里，回到主线程后直接调用就可以了

  避免使用 CALayer 的 Border、corner、shadow、mask 等技术，这些都会触发离屏渲染。

3. 异步绘制

4. 全局并发线程

5. 高效的图片异步加载




### 如果 tableview 的 cell 中的图片还未加载完就滑出屏幕外了，这种情况如何处理

从 UIScrollView 的角度出发，对 cell 进行按需加载，即滚动很快时候，只加载目标范围内的 cell。

```objc
if (needLoadArr.count>0 && [needLoadArr indexOfObject:indexPath]==NSNotFound) {
    [cell clear]; return;
}
```

例如：如果目标行与当前行相差超过指定行数，只在目标滚动范围的前后指定3行加载。

```objc
- (void)scrollViewWillEndDragging:(UIScrollView *)scrollView withVelocity:(CGPoint)velocity targetContentOffset:(inout CGPoint *)targetContentOffset{
    NSIndexPath *ip = [self indexPathForRowAtPoint:CGPointMake(0, targetContentOffset->y)];
    NSIndexPath *cip = [[self indexPathsForVisibleRows] firstObject];
    NSInteger skipCount = 8;
    if (labs(cip.row-ip.row)>skipCount) {
        NSArray *temp = [self indexPathsForRowsInRect:CGRectMake(0, targetContentOffset->y, self.width, self.height)];
        NSMutableArray *arr = [NSMutableArray arrayWithArray:temp];
        if (velocity.y<0) {
            NSIndexPath *indexPath = [temp lastObject];
            if (indexPath.row+33) {
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-3 inSection:0]];
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-2 inSection:0]];
                [arr addObject:[NSIndexPath indexPathForRow:indexPath.row-1 inSection:0]];
            }
        }
        [needLoadArr addObjectsFromArray:arr];
    }
}
```



<!-- ** -->

### 如何检测离屏渲染与优化

检测，通过勾选Xcode的Debug->View Debugging-->Rendering->Run->Color Offscreen-Rendered Yellow项。

优化，如阴影，在绘制时添加阴影的路径



### 怎么检测图层混合

1、模拟器debug中color blended layers红色区域表示图层发生了混合

2、Instrument-选中Core Animation-勾选Color Blended Layers

避免图层混合：

确保控件的opaque属性设置为true，确保backgroundColor和父视图颜色一致且不透明

如无特殊需要，不要设置低于1的alpha值

确保UIImage没有alpha通道

UILabel图层混合解决方法：

iOS8以后设置背景色为非透明色并且设置label.layer.masksToBounds=YES让label只会渲染她的实际size区域，就能解决UILabel的图层混合问题

iOS8 之前只要设置背景色为非透明的就行

为什么设置了背景色但是在iOS8上仍然出现了图层混合呢？

UILabel在iOS8前后的变化，在iOS8以前，UILabel使用的是CALayer作为底图层，而在iOS8开始，UILabel的底图层变成了_UILabelLayer，绘制文本也有所改变。在背景色的四周多了一圈透明的边，而这一圈透明的边明显超出了图层的矩形区域，设置图层的masksToBounds为YES时，图层将会沿着Bounds进行裁剪 图层混合问题解决了





<br>

> **参考资料**
> - [《iOS 保持界面流畅的技巧》 —— ibireme](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)
> - https://github.com/texturegroup/texture
> - https://github.com/johnil/VVeboTableViewDemo
> - https://github.com/forkingdog/UITableView-FDTemplateLayoutCell
> - https://github.com/facebook/AsyncDisplayKit/tree/master/examples/SocialAppLayout
