---
layout: post
title: "Core Animation Advanced Technique 学习笔记(3)"
date: 2014-07-30 20:30:51 +0800
comments: true
categories: iOS-CoreAnimation
---

#第一部分：下面的图层
##4.视觉效果（Visual Effects）

###4.1、圆角

CALayer有一个叫做`cornerRadius`的属性控制着图层角的曲率。

它是一个浮点数，默认为0（即直角，没有圆角）。

默认情况下，这个曲率值只影响该图层的背景颜色而不影响背景图片或是子图层。

不过，如果把masksToBounds设置成YES的话，图层里面的所有东西都会被截取。


**举个例子**

```
新建两个View,一个View背景色是白色，带有圆角，一个View背景色是红色，中心点在白色View的左上角上，这样会形成什么样的效果？
	
红色View会盖住一部分白色View

如果此时把白色View的masksToBounds设置为YES,会如何？
```

**下图很清晰地展现了结果，左边是masksToBounds设置为NO,右边是masksToBounds设置为YES**

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.1.png)


当然，单独控制每个层次的圆角也是可以做到的，而且我们经常会用到的上面两个角是圆角，下面两个角是直角的View或者Button的时候，就不能直接用`cornerRadius`了，而是需要用`图层蒙版`或者`CAShapeLayer`


###4.2、图层的边框

CALayer有两个经常用到的属性就是`borderWidth`和`borderColor`。这条线（也被称作stroke）沿着图层的bounds绘制（包含图层的角）。

`borderWidth`定义边框粗细,单位为点，浮点型，默认为0。

`borderColor`定义了边框的颜色，默认为黑色。它是CGColorRef类型，而不是UIColor，虽然属性声明并不能证明这一点，但图层引用了borderColor。CGColorRef在引用/释放时候的行为与NSObject极其相似。但是Objective-C语法并不支持这一做法，所以CGColorRef属性即便是强引用也只能声明成assign

**边框是绘制在图层边界里面的，而且在所有子内容和子图层之前。**

**如下图，设置borderWidth = 5.0f**

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.2.png)


边框并不会把其中包含的图或子图层的形状计算进来，如果图层的子图层超过了边界，或者是内置图在透明区域有一个透明蒙板，边框仍然会沿着图层的边界绘制出来


![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.3.png)



###4.3、阴影
####4.3.1阴影介绍

还有一个常见特性是阴影。阴影可以暗示图层深度。也能用来强调正在显示的图层和优先级，或者只是单纯的装饰。

只要设置`shadowOpacity`，给一个大于0（0是默认值）的数，就可以显示阴影了，可以显示在任意图层之下。

**注：shadowOpacity是一个必须在0.0（不可见）和1.0（完全不透明）之间的浮点数。**

如果设置为1.0，将会显示一个有轻微模糊的黑色阴影。要调整阴影还可以使用CALayer的另外三个属性：`shadowColor`，`shadowOffset`和`shadowRadius`。

`shadowColor`控制着阴影的颜色，类型是`CGColorRef`，默认为黑色


`shadowOffset`控制着阴影的方向和距离。类型是CGSize，width控制这阴影横向的位移，height控制着纵向的位移。默认值是 {0, -3}，即阴影相对于Y轴有3个点的向上位移。

**为什么阴影默认向上？**

	Core Animation是在Mac OS上面世的，而Mac OS和iOS上，二者的Y轴是颠倒的。
	
	这就导致了默认的3个点位移的阴影是向上的。
	
	在Mac上，shadowOffset的默认值是阴影向下的，因此iOS上的阴影方向是向上的了.

Mac和iOS上的阴影，左侧为iOS的，右侧为Mac的
![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.4.png)

苹果更倾向于让用户界面的阴影应该是垂直向下，所以在iOS把阴影宽度设为0，然后高度设为一个正值不失为一个做法。

`shadowRadius`属性控制着阴影的模糊度，当值是0时，阴影就和视图一样有一个非常确定的边界线。当值越来越大的时候，边界线看上去就会越来越模糊和自然。

通常情况，如果想让视图或控件非常醒目（比如弹出框遮罩层），就应该给shadowRadius设置一个稍大的值。阴影越模糊，图层的深度看上去就会更明显，如下图.

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.5.png)


####4.3.2、阴影裁剪

和图层边框不同，图层的阴影继承自内容的外形，而不是根据边界和角半径来确定。

为了计算出阴影的形状，Core Animation会将层中包含的图（包括子视图，如果有的话）考虑在内，然后通过这些来完美搭配图层形状从而创建一个阴影

如图

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.6.png)

**注：阴影通常就是在Layer的边界之外，于是会出现一些尴尬的效果**

如果masksToBounds设为YES，所有从图层中突出来的内容都会被会被剪掉。见下图

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.7.png)

阴影被剪掉了，如果想沿着内容裁切，就需要用到两个图层：一个只画阴影的空的外图层，和一个用masksToBounds裁剪内容的内图层。

**解决方案是：只把阴影应用在最外层的视图上，而内层视图进行裁剪**，如下图，这次阴影没有被减掉了。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.8.png)


####4.3.3、shadowPath

已知图层阴影并不总是方的，而是从图层内容的形状继承而来。

但是实时计算阴影也是一个非常消耗资源的，尤其是当图层有多个子图层，而且每个图层还有一个有透明效果的图的时候。（崩溃吧）

如果事先知道阴影形状会是什么样子的，则可以通过指定一个`shadowPath`来提高性能。

`shadowPath`是CGPathRef类型（一个指向CGPath的指针）。而CGPath是一个Core Graphics对象，用来指定任意的一个矢量图形。我们可以通过这个属性单独于图层形状之外指定阴影的形状。如图：

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.9.png)

```
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView1;
@property (nonatomic, weak) IBOutlet UIView *layerView2;
@end

@implementation ViewController

- (void)viewDidLoad
{
  [super viewDidLoad];

  //enable layer shadows
  self.layerView1.layer.shadowOpacity = 0.5f;
  self.layerView2.layer.shadowOpacity = 0.5f;

  //create a square shadow
  CGMutablePathRef squarePath = CGPathCreateMutable();
  CGPathAddRect(squarePath, NULL, self.layerView1.bounds);
  self.layerView1.layer.shadowPath = squarePath; CGPathRelease(squarePath);

  ￼//create a circular shadow
  CGMutablePathRef circlePath = CGPathCreateMutable();
  CGPathAddEllipseInRect(circlePath, NULL, self.layerView2.bounds);
  self.layerView2.layer.shadowPath = circlePath; CGPathRelease(circlePath);
}
@end
```

**注：如果是规则的图形，用CGPath很不错，如果是复杂形状的图形，则可以使用UIBezierPath更好**


###4.4、图层蒙版

有时候你希望展现的内容不是在一个矩形或圆角矩形。譬如一个有星形框架的图片，又或者想让一些古卷文字慢慢渐变成背景色，而不是一个突兀的边界。

创建一个不规则视图最方便的办法就是**使用一个32位有alpha通道的png图片**，然后给它指定一个透明蒙板。但是这个方法不能让你以编码的方式动态生成蒙板，也不能让子图层或子视图裁剪成同样的形状。

	CALayer有一个属性叫做`mask`可以解决这个问题。

`mask`这个属性是个CALayer类型，有和其他图层一样的绘制和布局属性。类似于一个子图层，相对于父图层（即拥有该属性的图层）布局，但是它却不是一个普通的子图层。不同于那些绘制在父图层中的子图层，mask图层定义了父图层的部分可见区域。

mask图层的颜色完全不重要，只有图层的`轮廓`最重要。mask属性就像是一个饼干切割机，mask图层实心的部分会被保留下来，其他的则会被抛弃。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.10.png)

**如果mask图层比父图层要小，只有在mask图层里面的内容才相关，除此以外的一切都会被隐藏起来**。


```
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIImageView *imageView;
@end

@implementation ViewController

- (void)viewDidLoad
{
  [super viewDidLoad];

  //create mask layer
  CALayer *maskLayer = [CALayer layer];
  maskLayer.frame = self.layerView.bounds;
  UIImage *maskImage = [UIImage imageNamed:@"Cone.png"];
  maskLayer.contents = (__bridge id)maskImage.CGImage;

  //apply mask to image layer￼
  self.imageView.layer.mask = maskLayer;
}
@end
```

设置好蒙版之后，效果如下

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.11.png)

**注：任何有图层的都可以作为蒙版，因此可以用代码甚至动画来生成，还是非常方便的**

###4.5、缩放过滤
####4.5.1、三种过滤算法

当视图显示一个图片的时候，都应该以正确的比例和正确的1：1像素显示在屏幕上。原因如下：

- 能够显示最好的画质，像素既没有被压缩也没有被拉伸。

- 能更好的使用内存。

- 最好的性能表现，CPU不需要为此额外的计算。

- 有时候，显示一个非真实大小的图片确实是我们需要的效果。比如说一个头像或是图片的缩略图，再比如说一个可以被拖拽和伸缩的大图。这些情况下，为同一图片的不同大小存储不同的图片显得又不切实际。

当图片需要显示不同的大小的时候，有一种叫做`缩放过滤`的算法就起到作用了。**它作用于原图的像素上并根据需要生成新的像素并显示在屏幕上**。

**CALayer为此提供了三种缩放过滤的方式**

```
kCAFilterLinear
kCAFilterNearest
kCAFilterTrilinear
```

`minification`（缩小图片）和`magnification`（放大图片）默认的过滤器都是`kCAFilterLinear`，这个过滤器采用`双线性滤波算法`，它在大多数情况下都表现良好。

`双线性滤波算法`通过对多个像素取样最终生成新的值，得到一个平滑的表现不错的拉伸。

**但是当放大倍数比较大的时候图片就模糊不清了。**

`kCAFilterTrilinear`和`kCAFilterLinear`非常相似，大部分情况下二者都看不出来有什么差别。

但是，较双线性滤波算法而言，`三线性滤波算法`存储了多个大小情况下的图片（也叫多重贴图），并三维取样，同时结合大图和小图的存储进而得到最后的结果。

这使得算法能够从一系列已经接近于最终大小的图片中得到想要的结果，换言之不需要对很多像素同步取样。这不仅提高了性能，也避免了因舍入错误引起的取样失灵的小概率问题

`kCAFilterNearest`是一种比较武断的方法。这个算法（也叫最近过滤）就是取样最近的单像素点而不管其他的颜色。这样做非常快，也不会使图片模糊。但是，最明显的效果就是，会使得压缩图片更糟，图片放大之后也显得块状或是马赛克严重。


下图是双线性滤波和三线性滤波与最近过滤算法之间的对比，主要对大图

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.12.png)

下图是没有斜线的小图，使用三种过滤算法的结果

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.13.png)

总的来说，对于比较小的图或者是差异特别明显，极少斜线的大图，最近过滤算法会保留这种差异明显的特质以呈现更好的结果。但是对于大多数的图尤其是有很多斜线或是曲线轮廓的图片来说，最近过滤算法会导致更差的结果。换句话说，**线性过滤保留了形状，最近过滤则保留了像素的差异**。

####4.5.2、小例子
我们用简单的像素字体创造数字显示方式，用图片存储起来，而且用之前介绍过的拼合技术来显示

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.14.png)

我们在Interface Builder中放置了六个视图，小时、分钟、秒钟各两个.用了一个IBOutletCollection对象把他们和控制器联系起来，这样就可以以数组的方式访问视图了，下图是代码，IB上放置控件是从左到右依次排列的。

```
@interface ViewController ()

@property (nonatomic, strong) IBOutletCollection(UIView) NSArray *digitViews;
@property (nonatomic, weak) NSTimer *timer;
￼￼
@end

@implementation ViewController

- (void)viewDidLoad
{
  [super viewDidLoad]; //get spritesheet image
  UIImage *digits = [UIImage imageNamed:@"Digits.png"];

  //set up digit views
  for (UIView *view in self.digitViews) {
    //set contents
    view.layer.contents = (__bridge id)digits.CGImage;
    view.layer.contentsRect = CGRectMake(0, 0, 0.1, 1.0);
    view.layer.contentsGravity = kCAGravityResizeAspect;
  }

  //start timer
  self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(tick) userInfo:nil repeats:YES];

  //set initial clock time
  [self tick];
}

- (void)setDigit:(NSInteger)digit forView:(UIView *)view
{
  //adjust contentsRect to select correct digit
  view.layer.contentsRect = CGRectMake(digit * 0.1, 0, 0.1, 1.0);
}

- (void)tick
{
  //convert time to hours, minutes and seconds
  NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier: NSGregorianCalendar];
  NSUInteger units = NSHourCalendarUnit | NSMinuteCalendarUnit | NSSecondCalendarUnit;
  ￼
  NSDateComponents *components = [calendar components:units fromDate:[NSDate date]];

  //set hours
  [self setDigit:components.hour / 10 forView:self.digitViews[0]];
  [self setDigit:components.hour % 10 forView:self.digitViews[1]];

  //set minutes
  [self setDigit:components.minute / 10 forView:self.digitViews[2]];
  [self setDigit:components.minute % 10 forView:self.digitViews[3]];

  //set seconds
  [self setDigit:components.second / 10 forView:self.digitViews[4]];
  [self setDigit:components.second % 10 forView:self.digitViews[5]];
}
@end
```
结果如下：

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.15.png)

因为默认是`kCAFilterLinear`，所以导致缩放图片时图片模糊了，在`for循环`中加入
	
	view.layer.magnificationFilter = kCAFilterNearest;

效果如下

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.16.png)


###4.6成组透明
UIView有一个`alpha`的属性来确定视图的`透明度`。CALayer有一个等同的属性叫做`opacity`，这两个属性都是影响子层级的。

**换言之，如果给一个图层设置了opacity属性，那它的子图层都会受此影响。**

iOS里经常会把控件的alpha值设置成0.5，即50%的不透明度，这样看起来仿佛是控件处于不可用的状态，但是当一个控件有子视图的时候就有点奇怪了

下图是一个内嵌了UILabel的自定义UIButton；左边是一个不透明的按钮，右边是50%透明度的相同按钮。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.17.png)

这个问题由透明度的混合叠加导致，当显示一个50%透明度的图层时，图层的每个像素都会一半显示图层的颜色，另一半显示图层下面的颜色。

但是如果图层包含一个同样显示50%透明的子图层时，你所看到的视图，50%来自子视图，25%来了图层本身的颜色，另外的25%则来自背景色。

在刚才的例子中，Button和Label都是白色背景。而且都是50%的可见度，于是叠加起来导致透明度度是75%，所以Label所在的区域看上去就没有周围的部分那么透明。

理想状况下，当你设置了一个图层的透明度，你希望它包含的整个图层树像一个整体一样的透明效果。

**方法一：你可以把Info.plist文件中的UIViewGroupOpacity设置为YES来达到这个效果**

**但是这个设置会影响到这个应用，整个app可能会受到不良影响。**

	如果UIViewGroupOpacity并未设置，iOS 6和以前的版本会默认为NO（也许以后的版本会有一些改变，还没研究ios7的效果如何）。

**方法二，可以设置CALayer的一个叫做shouldRasterize属性来实现组透明的效果**

如果它被设置为YES，在应用透明度之前，图层及其子图层都会被整合成一个整体的图片，这样就没有透明度混合的问题了

```
@interface ViewController ()
@property (nonatomic, weak) IBOutlet UIView *containerView;
@end

@implementation ViewController

- (UIButton *)customButton
{
  //create button
  CGRect frame = CGRectMake(0, 0, 150, 50);
  UIButton *button = [[UIButton alloc] initWithFrame:frame];
  button.backgroundColor = [UIColor whiteColor];
  button.layer.cornerRadius = 10;

  //add label
  frame = CGRectMake(20, 10, 110, 30);
  UILabel *label = [[UILabel alloc] initWithFrame:frame];
  label.text = @"Hello World";
  label.textAlignment = NSTextAlignmentCenter;
  [button addSubview:label];
  return button;
}

- (void)viewDidLoad
{
  [super viewDidLoad];

  //create opaque button
  UIButton *button1 = [self customButton];
  button1.center = CGPointMake(50, 150);
  [self.containerView addSubview:button1];

  //create translucent button
  UIButton *button2 = [self customButton];
  ￼
  button2.center = CGPointMake(250, 150);
  button2.alpha = 0.5;
  [self.containerView addSubview:button2];

  //enable rasterization for the translucent button
  button2.layer.shouldRasterize = YES;
  button2.layer.rasterizationScale = [UIScreen mainScreen].scale;
}
@end
```

为了打开`shouldRasterize`属性，要设置了图层的`rasterizationScale`属性。默认情况下，所有图层拉伸都是1.0， **所以如果你使用了`shouldRasterize`属性，你就要确保你设置了`rasterizationScale`属性去匹配屏幕，以防止出现Retina屏幕像素化的问题**。

**注：当shouldRasterize和UIViewGroupOpacity一起设置的时候，会出现性能问题**

下图是设置了shouldRasterize的效果

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced4.18.png)

