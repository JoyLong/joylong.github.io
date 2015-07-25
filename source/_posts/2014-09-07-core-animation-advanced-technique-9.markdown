---
layout: post
title: "Core Animation Advanced Technique 学习笔记(9)"
date: 2014-09-07 20:06:08 +0800
comments: true
categories: iOS-CoreAnimation
---

#第二部分：运动中的设置
##10、缓冲

###10.1、动画速率

动画实际上是一段时间内的变化，这就暗示了变化一定是随着某个特定的速率进行。

速率由以下公式计算而来：

	velocity = change / time

这里的变化可以指的是一个物体移动的距离，时间指动画持续的时长，但实际上动画应用于任意可以做动画的属性（譬如color和opacity）。

上面的等式假设了速率在整个动画过程中都是恒定不变的，对于这种恒定速率的动画我们称之为“线性滨化”，从技术的角度而言这是实现动画最简单的方式，但也是完全不真实的一种效果。

现实生活中的任何一个物体都会在运动中加速或者减速。

那么我们如何在动画中实现这种加速度呢？

Core Animation内嵌了一系列标准函数提供给我们使用。

####10.1.1、CAMediaTimingFunction

CAAnimation的timingFunction属性，是CAMediaTimingFunction类的一个对象。

如果想改变隐式动画的计时函数，同样也可以使用CATransaction的+setAnimationTimingFunction:方法。

要创建CAMediaTimingFunction，最简单的就是调用+timingFunctionWithName:的构造方法。传入如下几个常量之一：

- kCAMediaTimingFunctionLinear 
  
  线性的计时函数，同样也是CAAnimation的timingFunction属性为空时候的默认函数
  
- kCAMediaTimingFunctionEaseIn 

  慢慢加速然后突然停止。适用于自由落体等
  
- kCAMediaTimingFunctionEaseOut 

  全速开始，然后慢慢减速停止。它有一个削弱的效果，应用的场景比如一扇门慢慢地关上，而不是砰地一声
  
- kCAMediaTimingFunctionEaseInEaseOut

  慢慢加速然后再慢慢减速。这是现实世界大多数物体移动的方式，也是大多数动画来说最好的选择。
  
  如果只可以用一种缓冲函数的话，那就必须是它了。当使用UIView的动画方法时，默认是这个选项，但当创建CAAnimation的时候，就需要手动设置了。

- kCAMediaTimingFunctionDefault
	
  和kCAMediaTimingFunctionEaseInEaseOut很类似，但是加速和减速都稍微有些慢。
  
  和kCAMediaTimingFunctionEaseInEaseOut的区别很难察觉，可能是苹果觉得它对于隐式动画来说更适合（然后对UIKit就改变了想法，而是使用kCAMediaTimingFunctionEaseInEaseOut作为默认效果）
  
  虽说名字写是默认的，当创建显式的CAAnimation它并不是默认选项（换句话说，默认的图层行为动画用kCAMediaTimingFunctionDefault作为它们的计时方法）。
  

小例子：缓冲函数的简单测试(在运行之前改变缓冲函数的代码，然后点击任何地方来观察图层是如何通过指定的缓冲移动的)

	@interface ViewController ()

	@property (nonatomic, strong) CALayer *colorLayer;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建红色图层
    	self.colorLayer = [CALayer layer];
    	self.colorLayer.frame = CGRectMake(0, 0, 100, 100);
    	self.colorLayer.position = CGPointMake(self.view.bounds.size.width/2.0, self.view.bounds.size.height/2.0);
    	self.colorLayer.backgroundColor = [UIColor redColor].CGColor;
    	[self.view.layer addSublayer:self.colorLayer];
	}

	- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
	{
    	// 配置事务
    	[CATransaction begin];
    	[CATransaction setAnimationDuration:1.0];
    	[CATransaction setAnimationTimingFunction:[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut]];
    	// 设置位置
    	self.colorLayer.position = [[touches anyObject] locationInView:self.view];
    	// 提交事务
    	[CATransaction commit];
	}

	@end


####10.1.2、UIView动画缓冲

UIKit的动画也支持这些缓冲方法，只是语法和常量有些不同。

为了改变UIView动画的缓冲选项，给options参数添加如下常量之一：

	UIViewAnimationOptionCurveEaseInOut  // 默认值
	UIViewAnimationOptionCurveEaseIn 
	UIViewAnimationOptionCurveEaseOut 
	UIViewAnimationOptionCurveLinear


和CAMediaTimingFunction差不多，只是没有kCAMediaTimingFunctionDefault对应的值了。

小例子：使用UIKit动画的缓冲

	@interface ViewController ()

	@property (nonatomic, strong) UIView *colorView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建红色图层
    	self.colorView = [[UIView alloc] init];
    	self.colorView.bounds = CGRectMake(0, 0, 100, 100);
    	self.colorView.center = CGPointMake(self.view.bounds.size.width / 2, self.view.bounds.size.height / 2);
    	self.colorView.backgroundColor = [UIColor redColor];
    	[self.view addSubview:self.colorView];
	}

	- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
	{
    	// 实施动画
    	[UIView animateWithDuration:1.0 delay:0.0
                        	options:UIViewAnimationOptionCurveEaseOut
                     	animations:^{
                            	//set the position
                           	 	self.colorView.center = [[touches anyObject] locationInView:self.view];
                        	}
                     	completion:NULL];

	}

	@end


####10.1.3、缓冲和关键帧动画

之前讲到关键帧动画时：有个颜色切换的关键帧动画由于线性变换的原因看起来有些奇怪，使得颜色变换非常不自然。为了改善变换，可以用更加合适的缓冲方法，例如kCAMediaTimingFunctionEaseIn，给图层的颜色变化添加一点缓冲效果，让它更像现实中的一个彩色灯泡。

CAKeyframeAnimation有一个NSArray类型的timingFunctions属性，我们可以用它来对每次动画的步骤指定不同的计时函数。

但是指定函数的个数一定要等于keyframes数组的元素个数减一，因为它是描述每一帧`之间`动画速度的函数。

小例子：对CAKeyframeAnimation使用CAMediaTimingFunction

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *layerView;
	@property (nonatomic, weak) IBOutlet CALayer *colorLayer;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建子图层
   	 	self.colorLayer = [CALayer layer];
    	self.colorLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    	self.colorLayer.backgroundColor = [UIColor blueColor].CGColor;
    	// 添加到视图
    	[self.layerView.layer addSublayer:self.colorLayer];
	}

	- (IBAction)changeColor
	{
    	// 创建关键帧动画
	    CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    	animation.keyPath = @"backgroundColor";
    	animation.duration = 2.0;
    	animation.values = @[
        	                 (__bridge id)[UIColor blueColor].CGColor,
            	             (__bridge id)[UIColor redColor].CGColor,
                	         (__bridge id)[UIColor greenColor].CGColor,
               		      	 (__bridge id)[UIColor blueColor].CGColor ];
    	// 添加计时函数
    	CAMediaTimingFunction *fn = [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseIn];
    	animation.timingFunctions = @[fn, fn, fn];
    	// 应用动画到图层上
    	[self.colorLayer addAnimation:animation forKey:nil];
	}

	@end

###10.2、自定义缓冲函数

在现实世界中，钟表指针转动的时候，通常起步很慢，然后迅速啪地一声，最后缓冲到终点。

但是预设的缓冲函数没有适合这种方式的，那该如何创建一个新的呢？

除了+functionWithName:之外，CAMediaTimingFunction同样有另一个构造函数，一个有四个浮点参数的+functionWithControlPoints::::（注意这里奇怪的语法，并没有包含具体每个参数的名称，这在objective-C中是合法的，但是却违反了苹果对方法命名的指导方针，而且看起来是一个奇怪的设计）。

使用这个方法，我们可以创建一个自定义的缓冲函数，来匹配我们的时钟动画，为了理解如何使用这个方法，我们要了解一些CAMediaTimingFunction是如何工作的。

####10.2.1、三次贝塞尔曲线

CAMediaTimingFunction函数把输入的时间转换成起点和终点之间成比例的改变。

可以用一个简单的图标来解释，横轴代表时间，纵轴代表改变的量，于是线性的缓冲就是一条从起点开始的简单的斜线。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced10.1.png)

这条曲线的斜率代表了速度，斜率的改变代表了加速度，原则上来说，任何加速的曲线都可以用这种图像来表示，但是CAMediaTimingFunction使用了一个叫做三次贝塞尔曲线的函数，它只可以产出指定缓冲函数的子集。

一个三次贝塞尔曲线通过四个点来定义，第一个和最后一个点代表了曲线的起点和终点，剩下中间两个点叫做控制点，因为它们控制了曲线的形状，贝塞尔曲线的控制点其实是位于曲线之外的点，也就是说曲线并不一定要穿过它们。你可以把它们想象成吸引经过它们曲线的磁铁。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced10.2.png)

实际上它是一个很奇怪的函数，先加速，然后减速，最后快到达终点的时候又加速，那么标准的缓冲函数又该如何用图像来表示呢？

CAMediaTimingFunction有一个getControlPointAtIndex:values:的方法，可以用来检索曲线的点，使用它我们可以找到标准缓冲函数的点，然后用UIBezierPath和CAShapeLayer来把它画出来。

曲线的起始和终点始终是{0, 0}和{1, 1}，于是我们只需要检索曲线的第二个和第三个点（控制点）。

小例子：使用UIBezierPath绘制CAMediaTimingFunction

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *layerView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建计时函数
    	CAMediaTimingFunction *function = CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseOut];
    	// 获得控制点
    	CGPoint controlPoint1, controlPoint2;
    	[function getControlPointAtIndex:1 values:(float *)&controlPoint1];
    	[function getControlPointAtIndex:2 values:(float *)&controlPoint2];
    	// 创建曲线
    	UIBezierPath *path = [[UIBezierPath alloc] init];
    	[path moveToPoint:CGPointZero];
    	[path addCurveToPoint:CGPointMake(1, 1)
            controlPoint1:controlPoint1 controlPoint2:controlPoint2];
    	// 缩放路径到可显示状态
    	[path applyTransform:CGAffineTransformMakeScale(200, 200)];
    	// 创建CAShapeLayer
    	CAShapeLayer *shapeLayer = [CAShapeLayer layer];
    	shapeLayer.strokeColor = [UIColor redColor].CGColor;
    	shapeLayer.fillColor = [UIColor clearColor].CGColor;
    	shapeLayer.lineWidth = 4.0f;
    	shapeLayer.path = path.CGPath;
    	[self.layerView.layer addSublayer:shapeLayer];
    	// flip geometry so that 0,0 is in the bottom-left
    	self.layerView.layer.geometryFlipped = YES;
	}

	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced10.3.png)

那么对于我们自定义时钟指针的缓冲函数来说，我们需要初始微弱，然后迅速上升，最后缓冲到终点的曲线，通过一些实验之后，最终结果如下：

	[CAMediaTimingFunction functionWithControlPoints:1 :0 :0.75 :1];

如果把它转换成缓冲函数的图像，如下图：

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced10.4.png)

如果把它添加到时钟的程序，就形成了之前一直期待的非常赞的效果。

小例子：添加了自定义缓冲函数的时钟程序

	- (void)setAngle:(CGFloat)angle forHand:(UIView *)handView ￼animated:(BOOL)animated
	{
    	// 生成变换
    	CATransform3D transform = CATransform3DMakeRotation(angle, 0, 0, 1);
    	if (animated) {
        	// 创建变换动画
        	CABasicAnimation *animation = [CABasicAnimation animation];
        	animation.keyPath = @"transform";
        	animation.fromValue = [handView.layer.presentationLayer valueForKey:@"transform"];
        	animation.toValue = [NSValue valueWithCATransform3D:transform];
        	animation.duration = 0.5;
        	animation.delegate = self;
        	animation.timingFunction = [CAMediaTimingFunction functionWithControlPoints:1 :0 :0.75 :1];
        	// 应用动画
        	handView.layer.transform = transform;
        	[handView.layer addAnimation:animation forKey:nil];
   	 	} else {
        	// 直接设置变换
       	 	handView.layer.transform = transform;
    	}
	}

####10.2.2、更加复杂的动画缓冲

考虑一个橡胶球掉落到坚硬的地面的场景，当开始下落的时候，它会持续加速知道落到地面，然后经过几次反弹，最后停下来。如果用一张图来说明，如下图。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced10.5.png)

这种效果没法用一个简单的三次贝塞尔曲线表示，于是不能用CAMediaTimingFunction来完成。但如果想要实现这样的效果，可以用如下几种方法：

- 用CAKeyframeAnimation创建一个动画，然后分割成几个步骤，每个小步骤使用自己的计时函数。

- 使用定时器逐帧更新实现动画。

####10.2.3、基于关键帧的缓冲

为了使用关键帧实现反弹动画，我们需要在缓冲曲线中对每一个显著的点创建一个关键帧（在这个情况下，关键点也就是每次反弹的峰值），然后应用缓冲函数把每段曲线连接起来。同时，我们也需要通过keyTimes来指定每个关键帧的时间偏移，由于每次反弹的时间都会减少，于是关键帧并不会均匀分布。

小例子：使用关键帧实现反弹球的动画

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;
	@property (nonatomic, strong) UIImageView *ballView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 添加球的图片
    	UIImage *ballImage = [UIImage imageNamed:@"Ball.png"];
    	self.ballView = [[UIImageView alloc] initWithImage:ballImage];
    	[self.containerView addSubview:self.ballView];
    	// 开始动画
    	[self animate];
	}

	- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
	{
    	// 点击重播动画
    	[self animate];
	}

	- (void)animate
	{
    	// 重置球到屏幕顶部
    	self.ballView.center = CGPointMake(150, 32);
    	// 创建关键帧动画
    	CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    	animation.keyPath = @"position";
    	animation.duration = 1.0;
    	animation.delegate = self;
    	animation.values = @[
        	                 [NSValue valueWithCGPoint:CGPointMake(150, 32)],
            	             [NSValue valueWithCGPoint:CGPointMake(150, 268)],
                	         [NSValue valueWithCGPoint:CGPointMake(150, 140)],
                   		     [NSValue valueWithCGPoint:CGPointMake(150, 268)],
                   	    	 [NSValue valueWithCGPoint:CGPointMake(150, 220)],
                   	      	 [NSValue valueWithCGPoint:CGPointMake(150, 268)],
                         	 [NSValue valueWithCGPoint:CGPointMake(150, 250)],
                         	 [NSValue valueWithCGPoint:CGPointMake(150, 268)]
                         	];

    	animation.timingFunctions = @[
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseIn],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseOut],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseIn],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseOut],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseIn],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseOut],
                                  [CAMediaTimingFunction functionWithName: kCAMediaTimingFunctionEaseIn]
                                  ];

    	animation.keyTimes = @[@0.0, @0.3, @0.5, @0.7, @0.8, @0.9, @0.95, @1.0];
    	// 应用动画
    	self.ballView.layer.position = CGPointMake(150, 268);
    	[self.ballView.layer addAnimation:animation forKey:nil];
	}

	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced10.6.png)

这种方式还算不错，但是实现起来略显笨重（因为要不停地尝试计算各种关键帧和时间偏移）并且和动画强绑定了（因为如果要改变动画的一个属性，那就意味着要重新计算所有的关键帧）。那该如何写一个方法，用缓冲函数来把任何简单的属性动画转换成关键帧动画呢，下面我们来实现它。

####10.2.4、过程自动化

之前的例子中，把动画分割成相当大的几块，然后用Core Animation的缓冲进入和缓冲退出函数来大约形成我们想要的曲线。

但如果把动画分割成更小的几部分，那么就可以用直线来拼接这些曲线（也就是线性缓冲）。

为了实现自动化，我们需要知道如何做如下两件事情：

- 自动把任意属性动画分割成多个关键帧

- 用一个数学函数表示弹性动画，使得可以对帧做便宜

为了解决第一个问题，我们需要复制Core Animation的插值机制。

这是一个传入起点和终点，然后在这两个点之间指定时间点产出一个新点的机制。对于简单的浮点起始值，公式如下（假设时间从0到1）：

	value = (endValue – startValue) × time + startValue;

那么如果要插入一个类似于CGPoint，CGColorRef或者CATransform3D这种更加复杂类型的值，我们可以简单地对每个独立的元素应用这个方法（也就CGPoint中的x和y值，CGColorRef中的红，蓝，绿，透明值，或者是CATransform3D中独立矩阵的坐标）。

同样需要一些逻辑在插值之前对对象拆解值，然后在插值之后在重新封装成对象，也就是说需要实时地检查类型。

一旦我们可以用代码获取属性动画的起始值之间的任意插值，我们就可以把动画分割成许多独立的关键帧，然后产出一个线性的关键帧动画。

注意到我们用了60 x 动画时间（秒做单位）作为关键帧的个数，这时因为Core Animation按照每秒60帧去渲染屏幕更新，所以如果我们每秒生成60个关键帧，就可以保证动画足够的平滑（尽管实际上很可能用更少的帧率就可以达到很好的效果）。

小例子：使用插入的值创建一个关键帧动画

	float interpolate(float from, float to, float time)
	{
    	return (to - from) * time + from;
	}

	- (id)interpolateFromValue:(id)fromValue toValue:(id)toValue time:(float)time
	{
    	if ([fromValue isKindOfClass:[NSValue class]]) {
        	// 获得类型
        	const char *type = [fromValue objCType];
        	if (strcmp(type, @encode(CGPoint)) == 0) {
            	CGPoint from = [fromValue CGPointValue];
            	CGPoint to = [toValue CGPointValue];
            	CGPoint result = CGPointMake(interpolate(from.x, to.x, time), interpolate(from.y, to.y, time));
            	return [NSValue valueWithCGPoint:result];
        	}
    	}
    	// 提供安全的默认值
    	return (time < 0.5)? fromValue: toValue;
	}

	- (void)animate
	{
    	// 重置视图位置
    	self.ballView.center = CGPointMake(150, 32);
    	// 设置动画参数
    	NSValue *fromValue = [NSValue valueWithCGPoint:CGPointMake(150, 32)];
    	NSValue *toValue = [NSValue valueWithCGPoint:CGPointMake(150, 268)];
    	CFTimeInterval duration = 1.0;
    	// 生成关键帧
    	NSInteger numFrames = duration * 60;
    	NSMutableArray *frames = [NSMutableArray array];
    	for (int i = 0; i < numFrames; i++) {
        	float time = 1 / (float)numFrames * i;
        	[frames addObject:[self interpolateFromValue:fromValue toValue:toValue time:time]];
    	}
    	// 创建关键帧动画
    	CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    	animation.keyPath = @"position";
    	animation.duration = 1.0;
    	animation.delegate = self;
    	animation.values = frames;
    	// 应用动画
    	[self.ballView.layer addAnimation:animation forKey:nil];
	}
	
这可以起到作用，但效果并不是很好，到目前为止我们所完成的只是一个非常复杂的方式来使用线性缓冲复制CABasicAnimation的行为。

这种方式的好处在于我们可以更加精确地控制缓冲，这也意味着我们可以应用一个完全定制的缓冲函数。那么该如何做呢？

缓冲背后的数学并不很简单，但是幸运的是我们不需要一一实现它。罗伯特·彭纳有一个网页关于缓冲函数（http://www.robertpenner.com/easing），包含了大多数普遍的缓冲函数的多种编程语言的实现的链接，包括C。

这里是一个缓冲进入缓冲退出函数的示例（实际上有很多不同的方式去实现它）。

	float quadraticEaseInOut(float t) 
	{
    	return (t < 0.5)? (2 * t * t): (-2 * t * t) + (4 * t) - 1; 
	}

对我们的弹性球来说，我们可以使用bounceEaseOut函数：

	float bounceEaseOut(float t)
	{
    	if (t < 4/11.0) {
        	return (121 * t * t)/16.0;
    	} else if (t < 8/11.0) {
        	return (363/40.0 * t * t) - (99/10.0 * t) + 17/5.0;
    	} else if (t < 9/10.0) {
        	return (4356/361.0 * t * t) - (35442/1805.0 * t) + 16061/1805.0;
    	}
    	return (54/5.0 * t * t) - (513/25.0 * t) + 268/25.0;
	}

小例子：用关键帧实现自定义的缓冲函数

	- (void)animate
	{
    	// reset ball to top of screen
    	self.ballView.center = CGPointMake(150, 32);
    	// set up animation parameters
    	NSValue *fromValue = [NSValue valueWithCGPoint:CGPointMake(150, 32)];
    	NSValue *toValue = [NSValue valueWithCGPoint:CGPointMake(150, 268)];
    	CFTimeInterval duration = 1.0;
    	// generate keyframes
    	NSInteger numFrames = duration * 60;
    	NSMutableArray *frames = [NSMutableArray array];
    	for (int i = 0; i < numFrames; i++) {
        	float time = 1/(float)numFrames * i;
        	//apply easing
        	time = bounceEaseOut(time);
        	//add keyframe
        	[frames addObject:[self interpolateFromValue:fromValue toValue:toValue time:time]];
    	}
    	// create keyframe animation
    	CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    	animation.keyPath = @"position";
    	animation.duration = 1.0;
    	animation.delegate = self;
    	animation.values = frames;
    	// apply animation
    	[self.ballView.layer addAnimation:animation forKey:nil];
	}
