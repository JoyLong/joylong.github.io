---
layout: post
title: "Core Animation Advanced Technique 学习笔记(2)"
date: 2014-07-26 22:28:44 +0800
comments: true
categories: iOS-CoreAnimation
---
#第一部分：下面的图层
##3、图层几何学(Layer Geometry)

###3.1、布局属性

UIView有三个比较重要的布局属性：`frame`，`bounds`和`center`，这三者在CALayer中分别对应地`frame`，`bounds`和`position`。为了能清楚区分，图层用了“position”，视图用了“center”，但是他们都代表同样的值。

frame代表了图层的外部坐标（也就是在父图层上占据的空间）

bounds是内部坐标（{0, 0}通常是图层的左上角）

center和position都代表了相对于父图层anchorPoint所在的位置。

anchorPoint的属性将会在后续介绍到，现在把它想成图层的中心点就好了。

下图显示了这些属性是如何相互依赖的。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced3.1.jpeg)


###3.2、View和Layer的frame/bounds/center之间的关系

View的frame，bounds和center属性仅仅是存取方法，改变View的frame，实际上是在改变位于View下方CALayer的frame，不能够独立于Layer之外改变View的frame。

其实，frame其实是一个虚拟属性，是根据bounds，position和transform计算而来，所以当其中任何一个值发生改变，frame都会变化。相反，改变frame的值同样会影响到他们当中的值

当对Layer做变换的时候，比如旋转或者缩放，frame实际上代表了覆盖在Layer旋转之后的整个轴对齐的矩形区域，也就是说frame的宽高可能和bounds的宽高不再一致了

**注意下图的frame 和 bounds，这是旋转之后的View和Layer的属性情况**
![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced3.2.jpeg)

###3.3、锚点（anchorPoint）
View的center属性和Layer的position属性都指定了相对于父图层anchorPoint的位置。

图层的anchorPoint通过position来控制它的frame的位置。

默认来说，anchorPoint位于图层的中点，所以图层的将会以这个点为中心放置。

anchorPoint属性并没有被UIView接口暴露出来，这也是视图的position属性被叫做“center”的原因。

但是图层的anchorPoint可以被移动，比如你可以把它置于图层frame的左上角，于是图层的内容将会向右下角的position方向移动，而不是居中了。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced3.3.jpeg)



和contentsRect / contentsCenter属性类似，`anchorPoint`用`单位坐标`来描述，也就是图层的相对坐标，图层左上角是{0, 0}，右下角是{1, 1}，因此默认坐标是{0.5, 0.5}。anchorPoint可以通过指定x和y值小于0或者大于1，使它放置在图层范围之外。

上图中，当改变了anchorPoint，position属性保持固定的值并没有发生改变，但是frame却移动了。

####3.3.1、什么场合需要用到锚点？

举例说明，创建一个模拟闹钟的项目。

钟面和钟表由四张图片组成（图3.4），为了简单说明，我们还是用传统的方式来装载和加载图片，使用四个UIImageView实例（当然你也可以用正常的视图，设置他们图层的`contents`图片）。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced3.4.jpeg)

闹钟的组件通过IB来排列，这些图片视图嵌套在一个容器视图之内，并且自动调整和自动布局都被禁用了。这是因为自动调整会影响到视图的frame，当视图旋转的时候，frame是会发生改变的，这将会导致一些布局上的失灵。

我们用NSTimer来更新闹钟，使用视图的transform属性来旋转钟表

```
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIImageView *hourHand;
@property (nonatomic, weak) IBOutlet UIImageView *minuteHand;
@property (nonatomic, weak) IBOutlet UIImageView *secondHand;
@property (nonatomic, weak) NSTimer *timer;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //start timer
    self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(tick) userInfo:nil repeats:YES];
                  ￼
    //set initial hand positions
    [self tick];
}

- (void)tick
{
    //convert time to hours, minutes and seconds
    NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
    NSUInteger units = NSHourCalendarUnit | NSMinuteCalendarUnit | NSSecondCalendarUnit;
    NSDateComponents *components = [calendar components:units fromDate:[NSDate date]];
    CGFloat hoursAngle = (components.hour / 12.0) * M_PI * 2.0;
    //calculate hour hand angle //calculate minute hand angle
    CGFloat minsAngle = (components.minute / 60.0) * M_PI * 2.0;
    //calculate second hand angle
    CGFloat secsAngle = (components.second / 60.0) * M_PI * 2.0;
    //rotate hands
    self.hourHand.transform = CGAffineTransformMakeRotation(hoursAngle);
    self.minuteHand.transform = CGAffineTransformMakeRotation(minsAngle);
    self.secondHand.transform = CGAffineTransformMakeRotation(secsAngle);
}

@end

```

结果如下

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced3.5.jpeg)

你也许会认为可以在Interface Builder当中调整指针图片的位置来解决，但其实并不能达到目的，因为如果不放在钟面中间的话，同样不能正确的旋转。

也许在图片末尾添加一个透明空间也是个解决方案，但这样会让图片变大，也会消耗更多的内存，这样并不优雅。

更好的方案是使用anchorPoint属性，我们来在-viewDidLoad方法中添加几行代码来给每个钟指针的anchorPoint做一些平移

```
- (void)viewDidLoad 
{
    [super viewDidLoad];
    // adjust anchor points

    self.secondHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f); 
    self.minuteHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f); 
    self.hourHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f);


    // start timer
}
```

结果如下

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced3.6.jpeg)


###3.4、坐标系

和视图一样，图层在图层树当中也是相对于父图层按层级关系放置，一个图层的position依赖于它父图层的bounds，如果父图层发生了移动，它的所有子图层也会跟着移动。

这样你可以通过移动根图层来将它的子图层作为一个整体来移动，但是有时候你需要知道一个图层的绝对位置，或者是相对于另一个图层的位置，而不是它当前父图层的位置。

CALayer给不同坐标系之间的图层转换提供了一些工具类方法：

- (CGPoint)convertPoint:(CGPoint)point fromLayer:(CALayer *)layer; 
- (CGPoint)convertPoint:(CGPoint)point toLayer:(CALayer *)layer; 
- (CGRect)convertRect:(CGRect)rect fromLayer:(CALayer *)layer;
- (CGRect)convertRect:(CGRect)rect toLayer:(CALayer *)layer;

这些方法可以把定义在一个图层坐标系下的点或者矩形转换成另一个图层坐标系下的点或者矩形




###3.5、翻转的几何结构

常规说来，在iOS上，一个图层的position位于父图层的左上角，但是在Mac OS上，通常是位于左下角。

Core Animation可以通过`geometryFlipped`属性来适配这两种情况，它决定了一个图层的坐标是否相对于父图层垂直翻转，是一个BOOL类型。

在iOS上通过设置它为YES意味着它的子图层将会被垂直翻转，也就是将会沿着底部排版而不是通常的顶部（它的所有子图层也同理，除非把它们的geometryFlipped属性也设为YES）。


####3.5.1、Z坐标轴

和UIView严格的二维坐标系不同，CALayer存在于一个三维空间当中。

除了我们已经讨论过的position和anchorPoint属性之外，CALayer还有另外两个属性，`zPosition`和`anchorPointZ`，二者都是在Z轴上描述图层位置的浮点类型。

注意这里并没有更深的属性来描述由宽和高做成的bounds了，图层是一个完全扁平的对象，你可以把它们想象成类似于一页二维的坚硬的纸片，用胶水粘成一个空洞，就像三维结构的折纸一样。

zPosition属性在大多数情况下其实并不常用。

**除了做变换之外，zPosition最实用的功能就是改变图层的显示顺序了。**

通常，图层是根据它们子图层的sublayers出现的顺序来类绘制的，但是通过增加图层的zPosition，就可以把图层前置，于是它就在所有其他图层的前面了（或者至少是小于它的zPosition值的图层的前面）。

两个视图，绿色视图在红色后面，显示被红色遮住了一部分。


**如果我们提高绿色视图的zPosition**

```
- (void)viewDidLoad
{
    [super viewDidLoad];
    ￼
    //move the green view zPosition nearer to the camera
    self.greenView.layer.zPosition = 1.0f;
}
```

我们会发现红色在绿色后面了。其实并不需要增加太多，视图都非常地薄，给zPosition提高一个像素就可以让绿色视图前置，当然0.1或者0.0001也能够做到，但是最好不要这样，`因为浮点类型四舍五入的计算可能会造成一些不便的麻烦`。

###3.6、Layer处理事件

CALayer并不关心任何响应链事件，所以不能直接处理触摸事件或者手势。

但是它有一系列的方法帮你处理事件：`-containsPoint:`和`-hitTest:`。


####3.6.1、containsPoint

-containsPoint:接受一个在本图层坐标系下的CGPoint，如果这个点在图层frame范围内就返回YES。但是这需要把触摸坐标转换成每个图层坐标系下的坐标，结果很不方便。

下面是使用containsPoint判断被点击的图层

```
- (void)viewDidLoad
{
    [super viewDidLoad];
    //create sublayer
    self.blueLayer = [CALayer layer];
    self.blueLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    self.blueLayer.backgroundColor = [UIColor blueColor].CGColor;
    //add it to our view
    [self.layerView.layer addSublayer:self.blueLayer];
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    //get touch position relative to main view
    CGPoint point = [[touches anyObject] locationInView:self.view];
    //convert point to the white layer's coordinates
    point = [self.layerView.layer convertPoint:point fromLayer:self.view.layer];
    //get layer using containsPoint:
    if ([self.layerView.layer containsPoint:point]) {
        //convert point to blueLayer’s coordinates
        point = [self.blueLayer convertPoint:point fromLayer:self.layerView.layer];
        if ([self.blueLayer containsPoint:point]) {
            [[[UIAlertView alloc] initWithTitle:@"Inside Blue Layer" 
                                        message:nil
                                       delegate:nil 
                              cancelButtonTitle:@"OK"
                              otherButtonTitles:nil] show];
        } else {
            [[[UIAlertView alloc] initWithTitle:@"Inside White Layer"
                                        message:nil 
                                       delegate:nil
                              cancelButtonTitle:@"OK"
                              otherButtonTitles:nil] show];
        }
    }
}
```

####3.6.2、hitTest
-hitTest:方法同样接受一个CGPoint类型参数，而不是BOOL类型，它返回图层本身，或者包含这个坐标点的叶子节点图层。这意味着不再需要像使用-containsPoint:那样，人工地在每个子图层变换或者测试点击的坐标。如果这个点在最外面图层的范围之外，则返回nil。

```
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    //get touch position
    CGPoint point = [[touches anyObject] locationInView:self.view];
    //get touched layer
    CALayer *layer = [self.layerView.layer hitTest:point];
    //get layer using hitTest
    if (layer == self.blueLayer) {
        [[[UIAlertView alloc] initWithTitle:@"Inside Blue Layer"
                                    message:nil
                                   delegate:nil
                          cancelButtonTitle:@"OK"
                          otherButtonTitles:nil] show];
    } else if (layer == self.layerView.layer) {
        [[[UIAlertView alloc] initWithTitle:@"Inside White Layer"
                                    message:nil
                                   delegate:nil
                          cancelButtonTitle:@"OK"
                          otherButtonTitles:nil] show];
    }
}
```

注意当调用图层的-hitTest:方法时，测算的顺序严格依赖于图层树当中的图层顺序（和UIView处理事件类似）。zPosition属性可以明显改变屏幕上图层的顺序，但不能改变事件传递的顺序。

这意味着如果改变了图层的z轴顺序，你会发现将不能够检测到最前方的视图点击事件，这是因为被另一个图层遮盖住了，虽然它的zPosition值较小，但是在图层树中的顺序靠前。


###3.7、自动布局

你可能用过`UIViewAutoresizingMask`类型的一些常量，应用于当父视图改变尺寸的时候，相应UIView的frame也跟着更新的场景（通常用于横竖屏切换）。

在iOS6中，苹果介绍了自动排版机制，它和自动调整不同，并且更加复杂。

在Mac OS平台，CALayer有一个叫做layoutManager的属性可以通过CALayoutManager协议和CAConstraintLayoutManager类来实现自动排版的机制。但由于某些原因，这在iOS上并不适用。

当使用视图的时候，可以充分利用UIView类接口暴露出来的UIViewAutoresizingMask和NSLayoutConstraintAPI，但如果想随意控制CALayer的布局，就需要手工操作。

最简单的方法就是使用CALayerDelegate如下函数：

	- (void)layoutSublayersOfLayer:(CALayer *)layer;

当图层的bounds发生改变，或者图层的-setNeedsLayout方法被调用的时候，这个函数将会被执行。

这使得你可以手动地重新摆放或者重新调整子图层的大小，但是不能像UIView的autoresizingMask和constraints属性做到自适应屏幕旋转。

这也是为什么最好使用视图而不是单独的图层来构建应用程序的另一个重要原因之一。
