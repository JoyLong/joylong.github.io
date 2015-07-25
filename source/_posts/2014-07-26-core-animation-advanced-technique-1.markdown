---
layout: post
title: "Core Animation advanced Techniques 学习笔记(1)"
date: 2014-07-26 17:00:01 +0800
comments: true
categories: iOS-CoreAnimation
---

#第一部分：下面的图层
##1、图层树(The Layer Tree)
###图层介绍

在iOS当中，所有的视图都从一个叫做UIVIew的基类派生而来，UIView可以处理触摸事件，可以支持基于Core Graphics绘图，可以做仿射变换（例如旋转或者缩放），或者简单的类似于滑动或者渐变的动画。

和UIView最大的不同是CALayer不处理用户的交互。CALayer并不清楚具体的响应链（iOS通过视图层级关系用来传送触摸事件的机制），于是它并不能够响应事件，即使它提供了一些方法来判断是否一个触点在图层的范围之内

每一个UIView都有一个CALayer实例的图层属性，也就是所谓的backing layer，视图的职责就是创建并管理这个图层，以确保当子视图在层级关系中添加或者被移除的时候，他们关联的图层也同样对应在层级关系树当中有相同的操作

实际上这些背后关联的图层才是真正用来在屏幕上显示和做动画，UIView仅仅是对它的一个封装，提供了一些iOS类似于处理触摸的具体功能，以及Core Animation底层方法的高级接口。

但是为什么iOS要基于UIView和CALayer提供两个平行的层级关系呢？原因在于要做职责分离，这样也能避免很多重复代码。在iOS和Mac OS两个平台上，事件和用户交互有很多地方的不同，基于多点触控的用户界面和基于鼠标键盘有着本质的区别

这里有一些UIView没有暴露出来的CALayer的功能：

- 阴影，圆角，带颜色的边框
- 3D变换
- 非矩形范围
- 透明遮罩
- 多级非线性动画


###小例子：增加蓝色子图层

	CALayer *blueLayer = [CALayer layer];
	blueLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
	blueLayer.backgroundColor = [UIColor blueColor].CGColor;
	//把layer加到view上
	[self.layerView.layer addSublayer:blueLayer];

###为什么要使用Layer
一个UIView只有一个相关联的CALayer（自动创建），同时它也可以支持添加无数多个子CALayer，你可以显示创建一个单独的Layer，并且把它直接添加到视图关联图层的子Layer。尽管可以这样添加Layer，但往往我们只是见简单地处理View，他们关联的Layer并不需要额外地手动添加子Layer。

在Mac OS平台，10.8版本之前，一个显著的性能缺陷就是由于用了视图层级而不是单独在一个视图内使用CALayer树状层级。
但是在iOS平台，使用轻量级的UIView类并没有显著的性能影响（当然在Mac OS 10.8之后，NSView的性能同样也得到很大程度的提高）。

使用UIView而不是CALayer的好处在于，你能在使用所有CALayer底层特性的同时，也可以使用UIView的高级API（比如自动排版，布局和事件处理）。


然而，当满足以下条件的时候，你可能更需要使用CALayer而不是UIView
1.开发同时可以在Mac OS上运行的跨平台应用
2.使用多种CALayer的子类，并且不想创建额外的UIView
3.做一些对性能特别挑剔的工作，比如对UIView一些可忽略不计的操作都会引起显著的不同（尽管如此，你可能会直接想使用OpenGL绘图）



##2、图层中包含的图(The Backing Image)

###1.content
CALyer 有一个属性叫做contents，这个属性的类型被定义为id，意味着它可以是任何类型的对象。在这种情况下，你可以给contents属性赋任何值，你的app仍然能够编译通过。但是，在实践中，如果你给contents赋的不是CGImage，那么你得到的图层将是空白的。

**如果要给图层的寄宿图赋值，你可以按照以下这个方法：**

	layer.contents = (__bridge id)image.CGImage;



**把之前的例子修改一下**

	UIImage *image = [UIImage imageNamed:@"Snowman.png"];

	//直接给layer的contents赋值
	self.layerView.layer.contents = (__bridge id)image.CGImage;

**如下图**

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced2.1.png)

###2.contentGravity

我们加载的图片并不刚好是一个方的，为了适应这个视图，它有一点点被拉伸了。

在使用UIImageView的时候遇到过同样的问题，解决方法就是把contentMode属性设置成更合适的值，像这样：
	
	view.contentMode = UIViewContentModeScaleAspectFit;

不过UIView大多数视觉相关的属性比如contentMode，对这些属性的操作其实是对对应图层的操作。

CALayer与contentMode对应的属性叫做contentsGravity，但是它是一个NSString类型，而不是像对应的UIKit部分，那里面的值是枚举。contentsGravity可选的常量值有以下一些：

	kCAGravityCenter
	kCAGravityTop
	kCAGravityBottom
	kCAGravityLeft
	kCAGravityRight
	kCAGravityTopLeft
	kCAGravityTopRight
	kCAGravityBottomLeft
	kCAGravityBottomRight
	kCAGravityResize
	kCAGravityResizeAspect
	kCAGravityResizeAspectFill

和cotentMode一样，contentsGravity的目的是为了决定内容在图层的边界中怎么对齐，我们将使用kCAGravityResizeAspect，它的效果等同于UIViewContentModeScaleAspectFit， 同时它还能在图层中等比例拉伸以适应图层的边界。

	self.layerView.layer.contentsGravity = kCAGravityResizeAspect;

如下所示

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced2.2.png)


###3.ContentsScale

contentsScale属性定义了寄宿图的像素尺寸和视图大小的比例，默认情况下它是一个值为1.0的浮点数。

contentsScale并不是总会对屏幕上的寄宿图有影响。如果你尝试对我们的例子设置不同的值，你就会发现根本没任何影响。因为contents由于设置了contentsGravity属性，所以它已经被拉伸以适应图层的边界。

如果你只是单纯地想放大图层的contents图片，你可以通过使用图层的transform和affineTransform属性来达到这个目的（见第五章『Transforms』，里面对此有解释），这(指放大)也不是contengsScale的目的所在.

contentsScale属性其实属于支持高分辨率（又称Hi-DPI或Retina）屏幕机制的一部分。它用来判断在绘制图层的时候应该为寄宿图创建的空间大小，和需要显示的图片的拉伸度（假设并没有设置contentsGravity属性）。UIView有一个类似功能但是非常少用到的contentScaleFactor属性。

如果contentsScale设置为1.0，将会以每个点1个像素绘制图片，如果设置为2.0，则会以每个点2个像素绘制图片，这就是我们熟知的Retina屏幕。

这并不会对我们在使用kCAGravityResizeAspect时产生任何影响，因为它就是拉伸图片以适应图层而已，根本不会考虑到分辨率问题。但是如果我们把contentsGravity设置为kCAGravityCenter，那将会有很明显的变化（**如下所示**）

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced2.3.png)


我们的雪人不仅有点大还有点像素的颗粒感。那是因为和UIImage不同，CGImage没有拉伸的概念。当我们使用UIImage类去读取我们的雪人图片的时候，他读取了高质量的Retina版本的图片。但是当我们用CGImage来设置我们的图层的内容时，拉伸这个因素在转换的时候就丢失了。不过我们可以通过手动设置contentsScale来修复这个问题

	UIImage *image = [UIImage imageNamed:@"Snowman.png"]; //add it directly to our view's layer
  	self.layerView.layer.contents = (__bridge id)image.CGImage; //center the image
  	self.layerView.layer.contentsGravity = kCAGravityCenter;

  	//设置contentsScale以适应图片
  	self.layerView.layer.contentsScale = image.scale;

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced2.4.png)

当用代码的方式来处理寄宿图的时候，一定要记住要手动的设置图层的contentsScale属性，否则，你的图片在Retina设备上就显示得不正确啦。代码如下：

	layer.contentsScale = [UIScreen mainScreen].scale;



###4.maskToBounds


UIView有一个叫做clipsToBounds的属性可以用来决定是否显示超出边界的内容，CALayer对应的属性叫做masksToBounds，把它设置为YES，就不会显示超过图的范围了

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced2.5.png)

###5.contentsRect

CALayer的contentsRect属性允许我们在图层边框里显示寄宿图的一个子域。这涉及到图片是如何显示和拉伸的，所以要比contentsGravity灵活多了

和bounds，frame不同，contentsRect不是按点来计算的，它使用了单位坐标，单位坐标指定在0到1之间，是一个相对值（像素和点就是绝对值）。所以他们是相对与寄宿图的尺寸的。iOS使用了以下的坐标系统：

- 点 —— 在iOS和Mac OS中最常见的坐标体系。点就像是虚拟的像素，也被称作逻辑像素。在标准设备上，一个点就是一个像素，但是在Retina设备上，一个点等于2*2个像素。iOS用点作为屏幕的坐标测算体系就是为了在Retina设备和普通设备上能有一致的视觉效果。
- 像素 —— 物理像素坐标并不会用来屏幕布局，但是仍然与图片有相对关系。UIImage是一个屏幕分辨率解决方案，所以指定点来度量大小。但是一些底层的图片表示如CGImage就会使用像素，所以你要清楚在Retina设备和普通设备上，他们表现出来了不同的大小。
- 单位 —— 对于与图片大小或是图层边界相关的显示，单位坐标是一个方便的度量方式， 当大小改变的时候，也不需要再次调整。单位坐标在OpenGL这种纹理坐标系统中用得很多，Core Animation中也用到了单位坐标。
默认的contentsRect是{0, 0, 1, 1}，这意味着整个图默认都是可见的，如果我们指定一个小一点的矩形，图片就会被裁剪

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced2.6.png)

事实上给contentsRect设置一个负数的原点或是大于{1, 1}的尺寸也是可以的。这种情况下，最外面的像素会被拉伸以填充剩下的区域。

**contentsRect在app中最有趣的地方在于一个叫做image sprites（图片拼合）的用法。**

典型地，图片拼合后可以打包整合到一张大图上一次性载入。相比多次载入不同的图片，这样做能够带来很多方面的好处：内存使用，载入时间，渲染性能等等

首先 ，我们需要一个拼合后的图标--一个包含小一些的拼合图的大图片，如图

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced2.7.png)

接下来，像平常一样载入我们的大图，然后把它赋值给四个独立的图层的contents，然后设置每个图层的contentsRect来去掉我们不想显示的部分。

我们的工程中需要一些额外的视图。（为了避免太多代码。我们将使用Interface Builder来确定他们的位置，如果你愿意还是可以用代码的方式来实现的）。

	- (void)addSpriteImage:(UIImage *)image withContentRect:(CGRect)rect ￼toLayer:(CALayer *)layer //set image {
	
		layer.contents = (__bridge id)image.CGImage;

		//scale contents to fit
	  	layer.contentsGravity = kCAGravityResizeAspect;
	
	  	//set contentsRect
	  	layer.contentsRect = rect;
	}

	- (void)viewDidLoad  {
		[super viewDidLoad]; //load sprite sheet
	  	UIImage *image = [UIImage imageNamed:@"Sprites.png"];
	  	//set igloo sprite
	 	[self addSpriteImage:image withContentRect:CGRectMake(0, 0, 0.5, 0.5) toLayer:self.iglooView.layer];
	 	//set cone sprite
 	 	[self addSpriteImage:image withContentRect:CGRectMake(0.5, 0, 0.5, 0.5) toLayer:self.coneView.layer];
		//set anchor sprite
		[self addSpriteImage:image withContentRect:CGRectMake(0, 0.5, 0.5, 0.5) toLayer:self.anchorView.layer];
		//set spaceship sprite
		[self addSpriteImage:image withContentRect:CGRectMake(0.5, 0.5, 0.5, 0.5) toLayer:self.shipView.layer];
	}

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced2.8.png)

拼合不仅给app提供了一个整洁的载入方式，还有效地提高了载入性能（单张大图比多张小图载入地更快），但是如果有手动安排的话，他们还是有一些不方便的，如果你需要在一个已经创建好的品和图上做一些尺寸上的修改或者其他变动，无疑是比较麻烦的。

Mac上有一些商业软件可以为你自动拼合图片，这些工具自动生成一个包含拼合后的坐标的XML或者plist文件，拼合图片的使用大大简化。这个文件可以和图片一同载入，并给每个拼合的图层设置contentsRect，这样开发者就不用手动写代码来摆放位置了。

这些文件通常在OpenGL游戏中使用，不过呢，你要是有兴趣在一些常见的app中使用拼合技术，那么一个叫做LayerSprites的开源库（https://github.com/nicklockwood/LayerSprites)，它能够读取Cocos2D格式中的拼合图并在普通的Core Animation层中显示出来。

###6.contentsCenter

contentsCenter其实是一个CGRect，它定义了一个固定的边框和一个在图层上可拉伸的区域。 改变contentsCenter的值并不会影响到寄宿图的显示，除非这个图层的大小改变了，你才看得到效果。

默认情况下，contentsCenter是{0, 0, 1, 1}，这意味着如果大小（由conttensGravity决定）改变了,那么寄宿图将会均匀地拉伸开。但是如果我们增加原点的值并减小尺寸。我们会在图片的周围创造一个边框。下图展示了contentsCenter设置为{0.25, 0.25, 0.5, 0.5}的效果。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced2.9.png)

他工作起来的效果和UIImage里的-resizableImageWithCapInsets: 方法效果非常类似，只是它可以运用到任何寄宿图，甚至包括在Core Graphics运行时绘制的图形

	- (void)addStretchableImage:(UIImage *)image withContentCenter:(CGRect)rect toLayer:(CALayer *)layer
	{  
  		//set image
  		layer.contents = (__bridge id)image.CGImage;

  		//set contentsCenter
  		layer.contentsCenter = rect;
	}

	- (void)viewDidLoad
	{
  		[super viewDidLoad]; //load button image
  		UIImage *image = [UIImage imageNamed:@"Button.png"];

  		//set button 1
  		[self addStretchableImage:image withContentCenter:CGRectMake(0.25, 0.25, 0.5, 0.5) toLayer:self.button1.layer];

  		//set button 2
  		[self addStretchableImage:image withContentCenter:CGRectMake(0.25, 0.25, 0.5, 0.5) toLayer:self.button2.layer];
	}


###7.Custom Drawing

给contents赋CGImage的值不是唯一的设置寄宿图的方法。我们也可以直接用Core Graphics直接绘制寄宿图。能够通过继承UIView并实现-drawRect:方法来自定义绘制。

-drawRect: 方法没有默认的实现，如果UIView检测到-drawRect: 方法被调用了，它就会为视图分配一个寄宿图，这个寄宿图的像素尺寸等于视图大小乘以 contentsScale的值。

如果你不需要寄宿图，那就不要创建这个方法了，这会造成CPU资源和内存的浪费，这也是为什么苹果建议：如果没有自定义绘制的任务就不要在子类中写一个空的-drawRect:方法。

当视图在屏幕上出现的时候 -drawRect:方法就会被自动调用。-drawRect:方法里面的代码利用Core Graphics去绘制一个寄宿图，然后内容就会被缓存起来直到它需要被更新（通常是因为开发者调用了-setNeedsDisplay方法，尽管影响到表现效果的属性值被更改时，一些视图类型会被自动重绘，如bounds属性）。虽然-drawRect:方法是一个UIView方法，事实上都是底层的CALayer安排了重绘工作和保存了因此产生的图片。

CALayer有一个可选的delegate属性，实现了CALayerDelegate协议，当CALayer需要一个内容特定的信息时，就会从协议中请求。CALayerDelegate是一个非正式协议，其实就是说没有CALayerDelegate @protocol可以让你在类里面引用啦。你只需要调用你想调用的方法，CALayer会帮你做剩下的。（delegate属性被声明为id类型，所有的代理方法都是可选的）。

当需要被重绘时，CALayer会请求它的代理给他一个寄宿图来显示。它通过调用下面这个方法做到的:

	- (void)displayLayer:(CALayerCALayer *)layer;

如果代理想直接设置contents属性的话，它就可以这么做，不然没有别的方法可以调用了。如果代理不实现-displayLayer:方法，CALayer就会转而尝试调用下面这个方法：

	- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;
	
在调用这个方法之前，CALayer创建了一个合适尺寸的空寄宿图（尺寸由bounds和contentsScale决定）和一个Core Graphics的绘制上下文环境，为绘制寄宿图做准备，他作为ctx参数传入。


**小例子：实现CALayerDelegate做一些绘图工作**


	@implementation ViewController
	- (void)viewDidLoad
	{
  		[super viewDidLoad];
  ￼
  		//create sublayer
  		CALayer *blueLayer = [CALayer layer];
  		blueLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
  		blueLayer.backgroundColor = [UIColor blueColor].CGColor;

  		//set controller as layer delegate
  		blueLayer.delegate = self;

  		//ensure that layer backing image uses correct scale
  		blueLayer.contentsScale = [UIScreen mainScreen].scale; //add layer to our view
  		[self.layerView.layer addSublayer:blueLayer];

  		//force layer to redraw
  		[blueLayer display];
	}

	- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx
	{
  		//draw a thick red circle
  		CGContextSetLineWidth(ctx, 10.0f); 
  		CGContextSetStrokeColorWithColor(ctx, [UIColor redColor].CGColor);
  		CGContextStrokeEllipseInRect(ctx, layer.bounds);
	}
	@end


![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced2.10.png)

注意一下一些有趣的事情：

- 我们在blueLayer上显式地调用了-display。不同于UIView，当图层显示在屏幕上时，CALayer不会自动重绘它的内容。它把重绘的决定权交给了开发者。
- 尽管我们没有用masksToBounds属性，绘制的那个圆仍然沿边界被裁剪了。这是因为当你使用CALayerDelegate绘制寄宿图的时候，并没有对超出边界外的内容提供绘制支持。

现在你理解了CALayerDelegate，并知道怎么使用它。但是除非你创建了一个单独的图层，你几乎没有机会用到CALayerDelegate协议。因为当UIView创建了它的宿主图层时，它就会自动地把图层的delegate设置为它自己，并提供了一个-displayLayer:的实现，那所有的问题就都没了。

当使用寄宿了视图的图层的时候，你也不必实现-displayLayer:和-drawLayer:inContext:方法绘制你的寄宿图。通常做法是实现UIView的-drawRect:方法，UIView就会帮你做完剩下的工作，包括在需要重绘的时候调用-display方法。

