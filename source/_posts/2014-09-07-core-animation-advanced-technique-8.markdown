---
layout: post
title: "Core Animation Advanced Technique 学习笔记(8)"
date: 2014-09-07 16:06:08 +0800
comments: true
categories: iOS-CoreAnimation
---

#第二部分：运动中的设置
##9、图层时间

###9.1、CAMediaTiming协议

CAMediaTiming协议定义了在一段动画内用来控制已进行时间的属性的集合，CALayer和CAAnimation都实现了这个协议，所以时间可以被任意基于一个图层或者一段动画的类控制。

####9.1.1、持续和重复

CAMediaTiming有一个属性叫duration，duration是CFTimeInterval类型（和NSTimeInterval类似的双精度浮点类型）表示一次动画执行的时间

CAMediaTiming还有个属性叫做repeatCount，代表动画重复的执行次数。如果duration是2，repeatCount设为3.5，那么完整的动画时长将是7秒。

duration和repeatCount默认都是0。这里的0仅仅代表了“默认”，也就是0.25秒和1次

小例子：测试duration和repeatCount

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;
	@property (nonatomic, weak) IBOutlet UITextField *durationField;
	@property (nonatomic, weak) IBOutlet UITextField *repeatField;
	@property (nonatomic, weak) IBOutlet UIButton *startButton;
	@property (nonatomic, strong) CALayer *shipLayer;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 添加飞船
    	self.shipLayer = [CALayer layer];
    	self.shipLayer.frame = CGRectMake(0, 0, 128, 128);
    	self.shipLayer.position = CGPointMake(150, 150);
    	self.shipLayer.contents = (__bridge id)[UIImage imageNamed: @"Ship.png"].CGImage;
    	[self.containerView.layer addSublayer:self.shipLayer];
	}

	- (void)setControlsEnabled:(BOOL)enabled
	{
    	for (UIControl *control in @[self.durationField, self.repeatField, self.startButton]) {
        	control.enabled = enabled;
        	control.alpha = enabled? 1.0f: 0.25f;
    	}
	}
	
	- (IBAction)hideKeyboard
	{
    	￼[self.durationField resignFirstResponder];
    	[self.repeatField resignFirstResponder];
	}

	- (IBAction)start
	{
    	CFTimeInterval duration = [self.durationField.text doubleValue];
    	float repeatCount = [self.repeatField.text floatValue];
    	// 飞船旋转动画
    	CABasicAnimation *animation = [CABasicAnimation animation];
    	animation.keyPath = @"transform.rotation";
    	animation.duration = duration;
    	animation.repeatCount = repeatCount;
    	animation.byValue = @(M_PI * 2);
    	animation.delegate = self;
    	[self.shipLayer addAnimation:animation forKey:@"rotateAnimation"];
    	// 禁用控件
    	[self setControlsEnabled:NO];
	}

	- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag
	{
    	// 打开控件
    	[self setControlsEnabled:YES];
	}

	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced9.1.png)

创建重复动画的另一种方式是使用repeatDuration属性，它让动画重复一个指定的时间，而不是指定次数。

还可以设置autoreverses的属性（BOOL类型）在每次间隔交替循环过程中自动回放。

这对于播放一段连续非循环的动画很有用，例如打开一扇门，然后关上它，如下图：

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced9.2.png)

小例子：使用autoreverses属性实现门的摇摆（repeatDuration设置为INFINITY，会让动画无限循环播放，设置repeatCount为INFINITY也有同样的效果。但是repeatCount和repeatDuration可能会相互冲突。）

	@interface ViewController ()

	@property (nonatomic, weak) UIView *containerView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 添加一扇门
	    CALayer *doorLayer = [CALayer layer];
    	doorLayer.frame = CGRectMake(0, 0, 128, 256);
   	 	doorLayer.position = CGPointMake(150 - 64, 150);
    	doorLayer.anchorPoint = CGPointMake(0, 0.5);
    	doorLayer.contents = (__bridge id)[UIImage imageNamed: @"Door.png"].CGImage;
    	[self.containerView.layer addSublayer:doorLayer];
    	// 应用透视转换
    	CATransform3D perspective = CATransform3DIdentity;
    	perspective.m34 = -1.0 / 500.0;
    	self.containerView.layer.sublayerTransform = perspective;
    	// 应用摇摆的动画
    	CABasicAnimation *animation = [CABasicAnimation animation];
    	animation.keyPath = @"transform.rotation.y";
    	animation.toValue = @(-M_PI_2);
    	animation.duration = 2.0;
    	animation.repeatDuration = INFINITY;
    	animation.autoreverses = YES;
    	[doorLayer addAnimation:animation forKey:nil];
	}

	@end

####9.1.2、相对时间

时间都是相对的，每个动画都有它自己描述的时间，可以独立地加速，延时或者偏移。

beginTime指定了动画开始之前的的延迟时间。这里的延迟从动画添加到见到图层的那一刻开始测量，默认是0（即立刻执行动画）。

speed是一个时间的倍数，默认1.0，speed减少会减慢图层/动画的时间，speed增加会加快速度。如果2.0的速度，那么对于一个duration为1的动画，实际上在0.5秒的时候就已经完成了。

timeOffset和beginTime类似，但是和增加beginTime导致的延迟动画不同，增加timeOffset只是让动画快进到某一点，例如，对于一个持续1秒的动画来说，设置timeOffset为0.5意味着动画将从一半的地方开始。

和beginTime不同的是，timeOffset并不受speed的影响。所以如果你把speed设为2.0，把timeOffset设置为0.5，那么你的动画将从动画最后结束的地方开始，因为1秒的动画实际上被缩短到了0.5秒。然而即使使用了timeOffset让动画从结束的地方开始，它仍然播放了一个完整的时长，这个动画仅仅是循环了一圈，然后从头开始播放。

小例子：测试timeOffset和speed属性

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;
	@property (nonatomic, weak) IBOutlet UILabel *speedLabel;
	@property (nonatomic, weak) IBOutlet UILabel *timeOffsetLabel;
	@property (nonatomic, weak) IBOutlet UISlider *speedSlider;
	@property (nonatomic, weak) IBOutlet UISlider *timeOffsetSlider;
	@property (nonatomic, strong) UIBezierPath *bezierPath;
	@property (nonatomic, strong) CALayer *shipLayer;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建路径
    	self.bezierPath = [[UIBezierPath alloc] init];
    	[self.bezierPath moveToPoint:CGPointMake(0, 150)];
    	[self.bezierPath addCurveToPoint:CGPointMake(300, 150) controlPoint1:CGPointMake(75, 0) controlPoint2:CGPointMake(225, 300)];
    	// 用CAShapeLayer绘制路径
    	CAShapeLayer *pathLayer = [CAShapeLayer layer];
    	pathLayer.path = self.bezierPath.CGPath;
    	pathLayer.fillColor = [UIColor clearColor].CGColor;
    	pathLayer.strokeColor = [UIColor redColor].CGColor;
    	pathLayer.lineWidth = 3.0f;
    	[self.containerView.layer addSublayer:pathLayer];
    	// 添加飞船
    	self.shipLayer = [CALayer layer];
    	self.shipLayer.frame = CGRectMake(0, 0, 64, 64);
    	self.shipLayer.position = CGPointMake(0, 150);
    	self.shipLayer.contents = (__bridge id)[UIImage imageNamed: @"Ship.png"].CGImage;
    	[self.containerView.layer addSublayer:self.shipLayer];
    	// 设置初始值
    	[self updateSliders];
	}

	- (IBAction)updateSliders
	{
    	CFTimeInterval timeOffset = self.timeOffsetSlider.value;
    	self.timeOffsetLabel.text = [NSString stringWithFormat:@"%0.2f", imeOffset];
    	float speed = self.speedSlider.value;
    	self.speedLabel.text = [NSString stringWithFormat:@"%0.2f", speed];
	}

	- (IBAction)play
	{
    	// 创建关键帧动画
   	 	CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    	animation.keyPath = @"position";
    	animation.timeOffset = self.timeOffsetSlider.value;
    	animation.speed = self.speedSlider.value;
    	animation.duration = 1.0;
    	animation.path = self.bezierPath.CGPath;
    	animation.rotationMode = kCAAnimationRotateAuto;
    	animation.removedOnCompletion = NO;
    	[self.shipLayer addAnimation:animation forKey:@"slide"];
	}

	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced9.3.png)


####9.1.3、fillModel

对于beginTime非0的一段动画来说，会出现一个当动画添加到图层上但什么也没发生的状态。

类似的，removeOnCompletion被设置为NO的动画将会在动画结束的时候仍然保持之前的状态。

这就产生了一个问题，当动画开始之前和动画结束之后，被设置动画的属性将会是什么值呢？

一种可能是属性和动画在没被添加之前保持一致，也就是在模型图层定义的值

另一种可能是保持动画开始之前那一帧，或者动画结束之后的那一帧。这就是所谓的填充，因为动画开始和结束的值用来填充开始之前和结束之后的时间。

这种行为可以被CAMediaTiming的fillMode控制。fillMode是一个NSString类型，有四种常量：

	kCAFillModeForwards 
	kCAFillModeBackwards 
	kCAFillModeBoth 
	kCAFillModeRemoved
	
默认是kCAFillModeRemoved

当动画不再播放的时候就显示图层模型指定的值剩下的三种类型向前/向后/即向前又向后来填充动画状态，使得动画在开始前或者结束后仍然保持开始和结束那一刻的值。

这可以避免动画结束后闪变，但是，如果要解决这个问题需要把removeOnCompletion设置为NO，另外需要给动画添加一个非空的键，于是可以在不需要动画的时候把它从图层上移除。

###9.2、逐层时间

每个动画和图层在时间上都有它自己的层级概念。

对每个图层调整时间将会影响到它本身和子图层的动画，但不会影响到父图层。

另一个相似点是所有的动画都被按照层级来排列（CAAnimationGroup）。

对CALayer或者CAGroupAnimation调整duration和repeatCount/repeatDuration属性并不会影响到子动画。

但是beginTime，timeOffset和speed属性将会影响到子动画。

然而在层级关系中，beginTime指定了父图层开始动画（或者组合关系中的父动画）和对象将要开始自己动画之间的偏移。

调整CALayer和CAGroupAnimation的speed属性将会对动画以及子动画速度应用一个缩放的因子。

####9.2.1、全局时间和本地时间

CoreAnimation有一个全局时间的概念，也就是所谓的马赫时间（“马赫”实际上是iOS和Mac OS系统内核的命名）。

马赫时间在设备上所有进程都是全局的(在不同设备上不是),可以使用CACurrentMediaTime函数来访问马赫时间：

	CFTimeInterval time = CACurrentMediaTime();

这个函数返回的值其实不重要（它返回了设备自从上次启动后的秒数），但是可以对动画的时间测量提供了一个相对值。

当设备休眠的时候马赫时间会暂停，也就是所有的CAAnimations（基于马赫时间）同样也会暂停。

因此马赫时间对长时间测量并不有用。

比如用CACurrentMediaTime去更新一个实时闹钟就不合适了。（可以用[NSDate date]）。

每个CALayer和CAAnimation实例都有自己本地时间的概念，是根据父图层/动画层级关系中的beginTime，timeOffset和speed属性计算。

就和转换不同图层之间坐标关系一样，CALayer同样也提供了方法来转换不同图层之间的本地时间。如下：

	- (CFTimeInterval)convertTime:(CFTimeInterval)t fromLayer:(CALayer *)l; 
	- (CFTimeInterval)convertTime:(CFTimeInterval)t toLayer:(CALayer *)l;

当用来同步不同图层之间有不同的speed，timeOffset和beginTime的动画，这些方法会很有用。

####9.2.2、暂停, 回退和快进

当动画speed属性为0时，可以暂停动画，但是不能对正在进行的动画使用这个属性。

给图层添加一个CAAnimation实际上是给动画对象做了一个不可变的拷贝，所以对原始动画对象属性的改变对真实的动画并没有作用。

而且直接用-animationForKey:来检索图层正在进行的动画可以返回正确的动画对象，但是修改它的属性将会抛出异常。

如果移除图层正在进行的动画，图层将会`急速`返回动画之前的状态。但如果在动画移除之前拷贝呈现图层到模型图层，动画将会看起来暂停在那里。但是不好的地方在于之后就不能再恢复动画了。

有一个简单的方法是利用CAMediaTiming来暂停图层本身。

如果把图层的speed设置成0，它会暂停任何添加到图层上的动画。

类似的，设置speed大于1.0将会快进，设置成一个负值将会倒回动画。

通过增加主窗口图层的speed，可以暂停整个应用程序的动画。

可以在app delegate设置如下进行验证：

	self.window.layer.speed = 100;

你也可以通过这种方式来减速，但其实也可以在模拟器通过切换慢速动画来实现。

###9.3、手动动画

timeOffset可以让你手动控制动画进程，通过设置speed为0，可以禁用动画的自动播放，然后来使用timeOffset来来回显示动画序列。

这可以使得运用手势来手动控制动画变得很简单。

小例子：通过触摸手势手动控制动画

	@interface ViewController ()

	@property (nonatomic, weak) UIView *containerView;
	@property (nonatomic, strong) CALayer *doorLayer;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 添加个门
   	 	self.doorLayer = [CALayer layer];
    	self.doorLayer.frame = CGRectMake(0, 0, 128, 256);
    	self.doorLayer.position = CGPointMake(150 - 64, 150);
    	self.doorLayer.anchorPoint = CGPointMake(0, 0.5);
    	self.doorLayer.contents = (__bridge id)[UIImage imageNamed:@"Door.png"].CGImage;
    	[self.containerView.layer addSublayer:self.doorLayer];
    	// 应用透视变换
    	CATransform3D perspective = CATransform3DIdentity;
    	perspective.m34 = -1.0 / 500.0;
    	self.containerView.layer.sublayerTransform = perspective;
    	// 添加摇摆门用的手势
    	UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] init];
    	[pan addTarget:self action:@selector(pan:)];
    	[self.view addGestureRecognizer:pan];
    	// 暂停所有图层动画
    	self.doorLayer.speed = 0.0;
   	 	// 应用摇摆动画
    	CABasicAnimation *animation = [CABasicAnimation animation];
    	animation.keyPath = @"transform.rotation.y";
    	animation.toValue = @(-M_PI_2);
    	animation.duration = 1.0;
    	[self.doorLayer addAnimation:animation forKey:nil];
	}

	- (void)pan:(UIPanGestureRecognizer *)pan
	{
    	// 通过手势获得水平部分
    	CGFloat x = [pan translationInView:self.view].x;
    	// 转换成动画进度
		x /= 200.0f;
    	// 更新timeOffset
    	CFTimeInterval timeOffset = self.doorLayer.timeOffset;
    	timeOffset = MIN(0.999, MAX(0.0, timeOffset - x));
    	self.doorLayer.timeOffset = timeOffset;
    	// 重置手势
    	[pan setTranslation:CGPointZero inView:self.view];
	}

	@end
