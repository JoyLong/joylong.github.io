---
layout: post
title: "Core Animation Advanced Technique 学习笔记(7)"
date: 2014-09-07 9:25:40 +0800
comments: true
categories: iOS-CoreAnimation
---

#第二部分：运动中的设置
##8、显式动画（Implicit Animations）
###8.1、属性动画

属性动画作用于图层的某个单一属性，并指定了这个属性的一个目标值，或者动画进行过程中一连串关键点的值。

属性动画分为两种：基础动画和关键帧动画。

####8.1.1、基础动画

动画其实就是一段时间内发生的改变，最简单的形式就是从一个值改变到另一个值，这也是CABasicAnimation最主要的功能。

CABasicAnimation是CAPropertyAnimation的一个子类，CAPropertyAnimation同时也是Core Animation所有动画类型的抽象基类。

作为一个抽象类，CAAnimation本身并没有做多少工作，它提供了一个计时函数，一个委托（用于反映动画状态）以及一个removedOnCompletion的布尔值，用于标识动画是否该在结束后自动释放（默认YES，为了防止内存泄露）。

CAAnimation同时实现了一些协议，包括CAAction（允许CAAnimation的子类可以提供图层行为），以及CAMediaTiming。

CAPropertyAnimation通过指定动画的keyPath作用到某个单一属性上，CAAnimation通常应用于一个指定的CALayer，于是这里指的也就是一个图层的keyPath了。实际上它是一个关键路径，而不仅仅是属性的名称，这意味着动画不仅可以作用于图层本身的属性，而且还包含了它的子成员的属性，甚至是一些虚拟的属性。

CABasicAnimation继承于CAPropertyAnimation，并添加了如下属性：

	id fromValue  // 动画开始之前属性的值
	id toValue    // 动画结束之后属性的值
	id byValue    // 执行过程中改变的值

它们被定义成id类型而不是一些具体的类型是因为属性动画可以用作很多不同种的属性类型，包括数字类型，矢量，变换矩阵，甚至是颜色或者图片。

id类型可以包含任意由NSObject派生的对象，加入要对一些不直接从NSObject继承的属性类型做动画，则意味着需要把这些值用一个对象来封装，或者强转成一个对象，就像某些功能和Objective-C对象类似的Core Foundation类型。

小例子：用于CAPropertyAnimation的一些类型转换

类型 | 对象类型 | 转换的代码例子
------------ | ------------- | ------------
CGFloat | NSNumber  | id obj = @(float);
CGPoint | NSValue  | id obj = [NSValue valueWithCGPoint:point);
CGSize | NSValue  | id obj = [NSValue valueWithCGSize:size);
CGRect | NSValue  | id obj = [NSValue valueWithCGRect:rect);
CATransform3D | NSValue  | id obj = [NSValue valueWithCATransform3D:transform);
CGImageRef | id  | id obj = (__bridge id)imageRef;
CGColorRef | id  | id obj = (__bridge id)colorRef;


fromValue，toValue和byValue属性可以用很多种方式来组合，但为了防止冲突，不能一次性同时指定这三个值。

例如，如果指定了fromValue等于2，toValue等于4，byValue等于3，那么Core Animation就不知道结果到底是4（toValue）还是5（fromValue + byValue）了。

他们的用法在CABasicAnimation头文件中已经描述的很清楚了，所以在这里就不重复了。总的说来，就是只需要指定toValue或者byValue，剩下的值都可以通过上下文自动计算出来。

小例子：通过CABasicAnimation来设置图层背景色

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *layerView;
	@property (nonatomic, strong) IBOutlet CALayer *colorLayer;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建子图层
    	self.colorLayer = [CALayer layer];
    	self.colorLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    	self.colorLayer.backgroundColor = [UIColor blueColor].CGColor;
    	// 添加到视图上
    	[self.layerView.layer addSublayer:self.colorLayer];
	}

	- (IBAction)changeColor
	{
    	￼// 创建一个新的随机颜色
    	CGFloat red = arc4random() / (CGFloat)INT_MAX;
    	CGFloat green = arc4random() / (CGFloat)INT_MAX;
    	CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    	UIColor *color = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
    	// 创建一个基本动画
    	CABasicAnimation *animation = [CABasicAnimation animation];
    	animation.keyPath = @"backgroundColor";
    	animation.toValue = (__bridge id)color.CGColor;
    	// 在图层上设置动画
    	[self.colorLayer addAnimation:animation forKey:nil];
	}

	@end
	
运行程序，结果有点差强人意，点击按钮，的确可以使图层动画过渡到一个新的颜色，然动画结束之后又立刻变回原始值。

这是因为动画并没有改变图层的模型，而只是展现。一旦动画结束并从图层上移除之后，图层就立刻恢复到之前定义的外观状态。我们从没改变过backgroundColor属性，所以图层就返回到原始的颜色。

当之前在使用隐式动画的时候，实际上它就是用例子中CABasicAnimation来实现的。但是在那个例子中，我们通过设置属性来打开动画。在这里我们做了相同的动画，但是并没有设置任何属性的值（这也是为什么会立刻变回初始状态的原因）。

把动画设置成一个图层的行为（然后通过改变属性值来启动动画）是到目前为止同步属性值和动画状态最简单的方式了。

假设因为UIView关联的图层不能这么做动画，那么有两种可以更新属性值的方式：

	在动画开始之前或者动画结束之后。

动画之前改变属性的值是最简单的办法，但这意味着我们不能使用fromValue这么好的特性了，而且要手动将fromValue设置成图层当前的值。

于是在动画创建之前插入如下代码，就可以解决问题了

	animation.fromValue = (__bridge id)self.colorLayer.backgroundColor; 
	self.colorLayer.backgroundColor = color.CGColor;

这的确是可行的，但还是有些问题，如果这里已经正在进行一段动画，我们需要从表现图层那里去获得fromValue，而不是模型图层。

此外，由于图层并不是UIView关联的图层，需要用CATransaction来禁用隐式动画行为，否则默认的图层行为会干扰我们的显式动画

更新之后的代码如下：

	CALayer *layer = self.colorLayer.presentationLayer ?:
	self.colorLayer;
 	animation.fromValue = (__bridge id)layer.backgroundColor;
	[CATransaction begin];
	[CATransaction setDisableActions:YES];
	self.colorLayer.backgroundColor = color.CGColor;
	[CATransaction commit];

小例子：抽象出一个立刻恢复到原始状态的可复用函数

	- (void)applyBasicAnimation:(CABasicAnimation *)animation toLayer:(CALayer *)layer
	￼{

    	// 设置fromValue
    	animation.fromValue = [layer.presentationLayer ?: layer valueForKeyPath:animation.keyPath];
   	 	// 更新属性，注：只有toValue不为nil时这个值才有意义
    	[CATransaction begin];
    	[CATransaction setDisableActions:YES];
    	[layer setValue:animation.toValue forKeyPath:animation.keyPath];
    	[CATransaction commit];
    	// 图层上应用动画
    	[layer addAnimation:animation forKey:nil];
	}

	- (IBAction)changeColor
	{
    	// 创建新的随机色
    	CGFloat red = arc4random() / (CGFloat)INT_MAX;
    	CGFloat green = arc4random() / (CGFloat)INT_MAX;
    	CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    	UIColor *color = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
    	// 创建基础动画
    	CABasicAnimation *animation = [CABasicAnimation animation];
    	animation.keyPath = @"backgroundColor";
    	animation.toValue = (__bridge id)color.CGColor;
    	// 应用动画
    	[self applyBasicAnimation:animation toLayer:self.colorLayer];
	}

这种简单的实现方式通过toValue而不是byValue来处理动画，这已经是朝更好的解决方案迈出一大步了。你可以把它添加给CALaye作为一个分类，以方便更好地使用。

如果不在动画开始之前去更新目标属性，那么就只能在动画完全结束或者取消的时候更新它。这意味着我们需要精准地在动画结束之后，图层返回到原始值之前更新属性。那么该如何找到这个点呢？

####8.1.2、CAAnimationDelegate

使用隐式动画时，可以在CATransaction的完成块中检测到动画完成。

但是这种方式并不适用于显式动画，因为这里的动画和事务并没太多关联。

因此想要知道一个显式动画在何时结束，需要实现CAAnimationDelegate协议。

CAAnimationDelegate在任何头文件中都找不到，但是可以在CAAnimation头文件或者苹果开发者文档中找到相关函数。

小例子：动画完成之后修改图层的背景色（当更新属性的时候，我们需要设置一个新的事务，并且禁用图层行为。否则动画会发生两次，一个是因为显式的CABasicAnimation，另一次是因为隐式动画）

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建子图层
    	self.colorLayer = [CALayer layer];
    	self.colorLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    	self.colorLayer.backgroundColor = [UIColor blueColor].CGColor;
    	// 添加到视图中
    	[self.layerView.layer addSublayer:self.colorLayer];
	}

	- (IBAction)changeColor
	{
    	// 创建新的随机色
    	CGFloat red = arc4random() / (CGFloat)INT_MAX;
    	CGFloat green = arc4random() / (CGFloat)INT_MAX;
    	CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    	UIColor *color = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
    	// 创建基础动画
    	CABasicAnimation *animation = [CABasicAnimation animation];
    	animation.keyPath = @"backgroundColor";
    	animation.toValue = (__bridge id)color.CGColor;
    	animation.delegate = self;
    	// 在图层上应用图层
    	[self.colorLayer addAnimation:animation forKey:nil];
	}

	- (void)animationDidStop:(CABasicAnimation *)anim finished:(BOOL)flag
	{
    	// 设置backgroundColor属性来匹配动画toValue
    	[CATransaction begin];
    	[CATransaction setDisableActions:YES];
    	self.colorLayer.backgroundColor = (__bridge CGColorRef)anim.toValue;
    	[CATransaction commit];
	}

	@end
	
对CAAnimation而言，当有多个动画的时候，回调方法中无法区分哪个动画。

之前写过一个例子，通过简单地每秒更新指针的角度来实现一个钟，但如果指针动态地转向新的位置会更加真实。

这个效果不能通过隐式动画来实现，因为这些指针都是UIView的实例，所以图层的隐式动画都被禁用了。

我们可以通过UIView的动画方法来实现。但如果想更好地控制动画时间，使用显式动画会更好。

而使用CABasicAnimation来做动画可能会更加复杂，因为我们需要在-animationDidStop:finished:中检测指针状态（用于设置结束的位置）。

动画本身会作为一个参数传入委托的方法，很多人认为可以通过这个参数来比较，然后区分是什么动画，但实际上并不起作用，因为委托传入的动画参数是原始值的一个深拷贝，从而不是同一个值。

当使用-addAnimation:forKey:把动画添加到图层，这里有一个到目前为止我们都设置为nil的key参数。这里的键是-animationForKey:方法找到对应动画的唯一标识符，而当前动画的所有键都可以用animationKeys获取。如果我们对每个动画都关联一个唯一的键，就可以对每个图层循环所有键，然后调用-animationForKey:来比对结果。尽管这不是一个优雅的实现。

幸运的是，还有一种更加简单的方法。像所有的NSObject子类一样，CAAnimation实现了KVC协议，于是你可以用-setValue:forKey:和-valueForKey:方法来存取属性。

	但是CAAnimation有一个不同的性能：它更像一个NSDictionary，可以让你随意设置键值对，即使和你使用的动画类所声明的属性并不匹配。

这意味着你可以用任意类型标记动画。在这里，我们给UIView类型的指针添加的动画，所以可以简单地判断动画到底属于哪个视图，然后在委托方法中用这个信息正确地更新钟的指针

小例子：使用KVC标记动画

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
    	// 调整锚点
    	self.secondHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f);
    	self.minuteHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f);
    	self.hourHand.layer.anchorPoint = CGPointMake(0.5f, 0.9f);
    	// 开始计时器
    	self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(tick) userInfo:nil repeats:YES];
    	// 设置初始位置
    	[self updateHandsAnimated:NO];
	}

	- (void)tick
	{
    	[self updateHandsAnimated:YES];
	}

	- (void)updateHandsAnimated:(BOOL)animated
	{
    	// 转换时间格式
    	NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
    	NSUInteger units = NSHourCalendarUnit | NSMinuteCalendarUnit | NSSecondCalendarUnit;
    	NSDateComponents *components = [calendar components:units fromDate:[NSDate date]];
    	// 计算小时角度
    	CGFloat hourAngle = (components.hour / 12.0) * M_PI * 2.0;
    	// 计算分钟角度
    	CGFloat minuteAngle = (components.minute / 60.0) * M_PI * 2.0;
    	// 计算秒针角度
    	CGFloat secondAngle = (components.second / 60.0) * M_PI * 2.0;
    	// 旋转
    	[self setAngle:hourAngle forHand:self.hourHand animated:animated];
    	[self setAngle:minuteAngle forHand:self.minuteHand animated:animated];
    	[self setAngle:secondAngle forHand:self.secondHand animated:animated];
	}

	- (void)setAngle:(CGFloat)angle forHand:(UIView *)handView animated:(BOOL)animated
	{
    	// 生成transform
    	CATransform3D transform = CATransform3DMakeRotation(angle, 0, 0, 1);
    	if (animated) {
        	// 创建动画
        	CABasicAnimation *animation = [CABasicAnimation animation];
        	[self updateHandsAnimated:NO];
        	animation.keyPath = @"transform";
        	animation.toValue = [NSValue valueWithCATransform3D:transform];
        	animation.duration = 0.5;
        	animation.delegate = self;
        	[animation setValue:handView forKey:@"handView"];
        	[handView.layer addAnimation:animation forKey:nil];
    	} else {
        	// 直接设置transform
        	handView.layer.transform = transform;
    	}
	}

	- (void)animationDidStop:(CABasicAnimation *)anim finished:(BOOL)flag
	{
    	// 设置最终位置
    	UIView *handView = [anim valueForKey:@"handView"];
    	handView.layer.transform = [anim.toValue CATransform3DValue];
	}

现在这个例子可以成功识别出每个图层停止动画的时间，更新它的变换到一个新值，但是这个代码在模拟器上运行正常，在设备上运行时在-animationDidStop:finished:委托方法调用之前，指针会迅速返回到原始值。

问题在于回调方法在动画完成之前已经被调用了，而且不能保证发生在属性动画返回初始状态之前。

这个问题可以用fillMode属性来解决。

####8.1.3、关键帧动画

CAKeyframeAnimation是另一种UIKit没有暴露但功能强大的类。

和CABasicAnimation类似，CAKeyframeAnimation同样是CAPropertyAnimation的一个子类，它依然作用于单一属性，但和CABasicAnimation不同之处在于，它并不限制于设置一个起始和结束的值，而是可以根据一连串随意的值来做动画。

CAKeyframeAnimation提供了关键节点的帧，然后Core Animation在非关键位置对每帧之间进行自动处理。

小例子：使用CAKeyframeAnimation应用一系列颜色的变化

	- (IBAction)changeColor
	{
    	// 创建一个关键帧动画
    	CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    	animation.keyPath = @"backgroundColor";
    	animation.duration = 2.0;
    	animation.values = @[
                         (__bridge id)[UIColor blueColor].CGColor,
                         (__bridge id)[UIColor redColor].CGColor,
                         (__bridge id)[UIColor greenColor].CGColor,
                         (__bridge id)[UIColor blueColor].CGColor ];
    	// 应用动画到图层
    	[self.colorLayer addAnimation:animation forKey:nil];
	}

要注意：动画序列中开始和结束的颜色都是蓝色，这是因为CAKeyframeAnimation并不会自动把当前值作为第一帧（就像CABasicAnimation那样把fromValue设为nil）。

动画会在开始的时候突变到第一帧的值，然后在动画结束的时候突然恢复到原始的值。

所以为了动画的平滑特性，我们需要开始和结束的关键帧来匹配当前属性的值。

当然可以创建一个结束和开始值不同的动画，那样的话就需要在动画启动之前手动更新属性和最后一帧的值保持一致。

我们用duration属性把动画时间从默认的0.25秒增加到2秒，以便于动画做的不那么快。

运行这段代码会发现颜色不断循环，但效果有些奇怪。这是因为动画以一个恒定的步调在运行。当在每个动画之间过渡的时候并没有减速，这就产生了一个略微奇怪的效果，为了让动画看起来更自然，需要调整一下缓冲。

提供一个数组的值就可以按照颜色变化做动画，但是CAKeyframeAnimation有另一种方式去指定动画，就是使用CGPath类型的path属性来定义运动的序列来绘制动画。

以一个宇宙飞船沿着一个简单曲线的实例来举例。

为了创建路径，我们需要使用一个三次贝塞尔曲线，它是一种使用开始点，结束点和另外两个控制点来定义形状的曲线，可以通过使用一个基于C的Core Graphics绘图指令来创建，但是用UIKit提供的UIBezierPath类会更简单。

这次用CAShapeLayer来绘制曲线。绘制完CGPath之后，我们用它来创建一个CAKeyframeAnimation，然后应用到我们的宇宙飞船。

小例子：沿着一个贝塞尔曲线对图层做动画

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建path
	    UIBezierPath *bezierPath = [[UIBezierPath alloc] init];
    	[bezierPath moveToPoint:CGPointMake(0, 150)];
	    [bezierPath addCurveToPoint:CGPointMake(300, 150) controlPoint1:CGPointMake(75, 0) controlPoint2:CGPointMake(225, 300)];
    	// 用CAShapeLayer绘制path
	    CAShapeLayer *pathLayer = [CAShapeLayer layer];
    	pathLayer.path = bezierPath.CGPath;
    	pathLayer.fillColor = [UIColor clearColor].CGColor;
    	pathLayer.strokeColor = [UIColor redColor].CGColor;
    	pathLayer.lineWidth = 3.0f;
    	[self.containerView.layer addSublayer:pathLayer];
    	// 添加飞船
	    CALayer *shipLayer = [CALayer layer];
    	shipLayer.frame = CGRectMake(0, 0, 64, 64);
    	shipLayer.position = CGPointMake(0, 150);
    	shipLayer.contents = (__bridge id)[UIImage imageNamed: @"Ship.png"].CGImage;
    	[self.containerView.layer addSublayer:shipLayer];
    	// 创建关键帧动画
	    CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    	animation.keyPath = @"position";
    	animation.duration = 4.0;
    	animation.path = bezierPath.CGPath;
    	[shipLayer addAnimation:animation forKey:nil];
	}

	@end
![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced8.1.png)

运行代码，会发现飞船的动画有些不太真实，这是因为当它运动的时候永远指向右边，而不是指向曲线切线的方向。有种方法是调整它的affineTransform来对运动方向做动画，但很可能和其它的动画冲突。

还有一种更加合适的方式就是使用CAKeyFrameAnimation的rotationMode属性。设置它为常量kCAAnimationRotateAuto，图层将会根据曲线的切线自动旋转。

小例子：通过rotationMode自动对齐图层到曲线

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建路径
    	...
    	// 创建关键帧动画
    	CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
    	animation.keyPath = @"position";
    	animation.duration = 4.0;
    	animation.path = bezierPath.CGPath;
    	animation.rotationMode = kCAAnimationRotateAuto;
    	[shipLayer addAnimation:animation forKey:nil];
	}

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced8.2.png)
####8.1.4、虚拟属性

如果想要对一个物体做旋转的动画，那就需要作用于transform属性，因为CALayer没有显式提供角度或者方向之类的属性

小例子：用transform属性对图层做动画

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 添加飞船
    	CALayer *shipLayer = [CALayer layer];
    	shipLayer.frame = CGRectMake(0, 0, 128, 128);
    	shipLayer.position = CGPointMake(150, 150);
    	shipLayer.contents = (__bridge id)[UIImage imageNamed: @"Ship.png"].CGImage;
    	[self.containerView.layer addSublayer:shipLayer];
    	// 飞船旋转
    	CABasicAnimation *animation = [CABasicAnimation animation];
    	animation.keyPath = @"transform";
    	animation.duration = 2.0;
    	animation.toValue = [NSValue valueWithCATransform3D: CATransform3DMakeRotation(M_PI, 0, 0, 1)];
    	[shipLayer addAnimation:animation forKey:nil];
	}

	@end

如果我们把旋转的值从M_PI（180度）调整到2 * M_PI（360度），然后运行程序，会发现这时候飞船完全不动了。这是因为这里的矩阵做了一次360度的旋转，和做了0度是一样的，所以最后的值根本没变。

现在继续使用M_PI，但这次用byValue而不是toValue。也许你会认为这和设置toValue结果一样，因为0 + 90度 == 90度，但实际上飞船的图片变大了，并没有做任何旋转，这是因为变换矩阵不能像角度值那样叠加。

如果需要独立于角度之外单独对平移或者缩放做动画呢？由于都需要我们来修改transform属性，实时地重新计算每个时间点的每个变换效果，然后根据这些创建一个复杂的关键帧动画，这一切都是为了对图层的一个独立做一个简单的动画。

幸运的是，有一个更好的解决方案：

为了旋转图层，我们可以对transform.rotation关键路径应用动画，而不是transform本身。

小例子：对虚拟的transform.rotation属性做动画

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 添加飞船
    	CALayer *shipLayer = [CALayer layer];
    	shipLayer.frame = CGRectMake(0, 0, 128, 128);
    	shipLayer.position = CGPointMake(150, 150);
    	shipLayer.contents = (__bridge id)[UIImage imageNamed: @"Ship.png"].CGImage;
    	[self.containerView.layer addSublayer:shipLayer];
    	// 飞船旋转
    	CABasicAnimation *animation = [CABasicAnimation animation];
    	animation.keyPath = @"transform.rotation";
    	animation.duration = 2.0;
    	animation.byValue = @(M_PI * 2);
    	[shipLayer addAnimation:animation forKey:nil];
	}

	@end

运行效果非常好。

用transform.rotation而不是transform做动画的好处如下：

- 我们可以不通过关键帧一步旋转多于180度的动画。

- 可以用相对值而不是绝对值旋转（设置byValue而不是toValue）。

- 可以不用创建CATransform3D，而是使用一个简单的数值来指定角度。

- 不会和transform.position或者transform.scale冲突（同样是使用关键路径来做独立的动画属性）。

transform.rotation属性其实并不存在。因为CATransform3D实际上是一个结构体，也没有符合KVC相关属性，transform.rotation实际上是CALayer用于处理动画变换的一个虚拟属性。

你不可以直接设置transform.rotation或者transform.scale，他们不能被直接使用。当你对他们做动画时，Core Animation自动地根据通过CAValueFunction来计算的值来更新transform属性。

CAValueFunction用于把我们赋给transform.rotation的简单浮点值转换成真正的用于摆放图层的CATransform3D矩阵值。你可以通过设置CAPropertyAnimation的valueFunction属性来改变，于是你设置的函数将会覆盖默认的函数。

CAValueFunction看起来似乎是对那些不能简单相加的属性（例如变换矩阵）做动画的非常有用的机制，但由于CAValueFunction的实现细节是私有的，所以目前不能通过继承它来自定义。赋值只能通过使用苹果目前已近提供的常量。

###8.2、动画组

CABasicAnimation和CAKeyframeAnimation仅仅作用于单独的属性

而CAAnimationGroup可以把这些动画组合在一起。

CAAnimationGroup是另一个继承于CAAnimation的子类，它添加了一个animations数组的属性，用来组合别的动画。

小例子：组合关键帧动画和基础动画

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建路径
    	UIBezierPath *bezierPath = [[UIBezierPath alloc] init];
    	[bezierPath moveToPoint:CGPointMake(0, 150)];
    	[bezierPath addCurveToPoint:CGPointMake(300, 150) controlPoint1:CGPointMake(75, 0) controlPoint2:CGPointMake(225, 300)];
    	// 用CAShapeLayer绘制路径
    	CAShapeLayer *pathLayer = [CAShapeLayer layer];
    	pathLayer.path = bezierPath.CGPath;
    	pathLayer.fillColor = [UIColor clearColor].CGColor;
    	pathLayer.strokeColor = [UIColor redColor].CGColor;
    	pathLayer.lineWidth = 3.0f;
    	[self.containerView.layer addSublayer:pathLayer];
    	// 添加带色图层
    	CALayer *colorLayer = [CALayer layer];
    	colorLayer.frame = CGRectMake(0, 0, 64, 64);
    	colorLayer.position = CGPointMake(0, 150);
    	colorLayer.backgroundColor = [UIColor greenColor].CGColor;
    	[self.containerView.layer addSublayer:colorLayer];
    	// 创建位置动画
    	CAKeyframeAnimation *animation1 = [CAKeyframeAnimation animation];
    	animation1.keyPath = @"position";
    	animation1.path = bezierPath.CGPath;
    	animation1.rotationMode = kCAAnimationRotateAuto;
    	// 创建颜色动画
    	CABasicAnimation *animation2 = [CABasicAnimation animation];
    	animation2.keyPath = @"backgroundColor";
    	animation2.toValue = (__bridge id)[UIColor redColor].CGColor;
    	// 创建动画组
    	CAAnimationGroup *groupAnimation = [CAAnimationGroup animation];
    	groupAnimation.animations = @[animation1, animation2]; 
    	groupAnimation.duration = 4.0;
    	// 添加到图层
    	[colorLayer addAnimation:groupAnimation forKey:nil];
	}

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced8.3.png)

###8.3、过渡

属性动画只对图层的可动画属性起作用，所以如果要改变一个不能动画的属性（比如图片），或者从层级关系中添加或者移除图层，属性动画将不起作用。

于是就有了过渡。过渡并不像属性动画那样平滑地在两个值之间做动画，而是影响到整个图层的变化。过渡动画首先展示之前的图层外观，然后通过一个交换过渡到新的外观。

为了创建一个过渡动画，我们将使用CATransition，CATransition同样是另一个CAAnimation的子类，和别的子类不同，CAAnimation有一个type和subtype来标识变换效果。type属性是一个NSString类型，可以被设置成如下类型：

	kCATransitionFade  // 默认类型，当你在改变图层属性之后，就创建了一个平滑的淡入淡出效果
	kCATransitionMoveIn // 顶部滑动进入，但不像推送动画那样把老图层推走
	kCATransitionPush  // 创建了一个新图层，从边缘的一侧滑动进来，把旧图层从另一侧推出去
	kCATransitionReveal // 把原始的图层滑动出去来显示新的外观，而不是把新的图层滑动进入

后面三种类型都有一个默认的动画方向，即都从左侧滑入

但是你可以通过subtype来控制它们的方向，提供了如下四种类型：

	kCATransitionFromRight 
	kCATransitionFromLeft 
	kCATransitionFromTop 
	kCATransitionFromBottom

小例子：使用CATransition来对UIImageView做动画

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIImageView *imageView;
	@property (nonatomic, copy) NSArray *images;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	//set up images
    	self.images = @[[UIImage imageNamed:@"Anchor.png"],
                    	[UIImage imageNamed:@"Cone.png"],
                    	[UIImage imageNamed:@"Igloo.png"],
                    	[UIImage imageNamed:@"Spaceship.png"]];
	}


	- (IBAction)switchImage
	{
    	// 设置变换类型
    	CATransition *transition = [CATransition animation];
    	transition.type = kCATransitionFade;
    	// 应用变换到imageView的寄宿图层上
    	[self.imageView.layer addAnimation:transition forKey:nil];
    	// 循环到下一张
    	UIImage *currentImage = self.imageView.image;
    	NSUInteger index = [self.images indexOfObject:currentImage];
    	index = (index + 1) % [self.images count];
    	self.imageView.image = self.images[index];
	}

	@end
	
过渡动画和之前的属性动画或者动画组添加到图层上的方式一致，都是通过-addAnimation:forKey:方法。

但是和属性动画不同的是，对指定的图层一次只能使用一次CATransition，因此，无论你对动画的键设置什么值，过渡动画都会对它的键设置成“transition”，也就是常量kCATransition。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced8.4.png)

####8.3.1、隐式过渡

CATransision可以对图层任何变化平滑过渡，这使得它为不好做动画的属性图层能够很好地提供动画支持。

当设置了CALayer的content属性的时候，CATransition的确是默认的行为。

但是对于视图关联的图层，或者是其他隐式动画的行为，这个特性依然是被禁用的，但是对于你自己创建的图层，这意味着对图层contents图片做的改动都会自动附上淡入淡出的动画。

####8.3.2、驱动图层树改变

CATransition并不作用于指定的图层属性，例如，在不知道UITableView哪一行被添加或者删除的情况下，直接就可以平滑地刷新它，或者在不知道UIViewController内部的视图层级的情况下对两个不同的实例做过渡动画。

但是要确保CATransition添加到的图层在过渡动画发生时不会在树状结构中被移除，否则CATransition将会和图层一起被移除。一般来说，你只需要将动画添加到被影响图层的superlayer。

小例子：为UITabBarController切换时做动画

	#import "AppDelegate.h"
	#import "FirstViewController.h" 
	#import "SecondViewController.h"
	#import <QuartzCore/QuartzCore.h>

	@implementation AppDelegate
	
	- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
	{
    	self.window = [[UIWindow alloc] initWithFrame: [[UIScreen mainScreen] bounds]];
    	UIViewController *viewController1 = [[FirstViewController alloc] init];
    	UIViewController *viewController2 = [[SecondViewController alloc] init];
    	self.tabBarController = [[UITabBarController alloc] init];
    	self.tabBarController.viewControllers = @[viewController1, viewController2];
    	self.tabBarController.delegate = self;
    	self.window.rootViewController = self.tabBarController;
    	[self.window makeKeyAndVisible];
    	return YES;
	}
	
	- (void)tabBarController:(UITabBarController *)tabBarController didSelectViewController:(UIViewController *)viewController
	{
    	￼// 设置淡入淡出变换
    	CATransition *transition = [CATransition animation];
    	transition.type = kCATransitionFade;
    	// 应用到tabBarController的视图上
    	[self.tabBarController.view.layer addAnimation:transition forKey:nil];
	}
	@end

####8.3.3、自定义过渡

CATransition只提供了四种默认动画类型，太少了。

但是苹果通过UIView +transitionFromView:toView:duration:options:completion:和+transitionWithView:duration:options:animations:方法提供了Core Animation的过渡特性。

但是这里的可用的过渡选项和CATransition的type属性提供的常量完全不同。UIView过渡方法中options参数可以由如下常量指定：

	UIViewAnimationOptionTransitionFlipFromLeft 
	UIViewAnimationOptionTransitionFlipFromRight
	UIViewAnimationOptionTransitionCurlUp 
	UIViewAnimationOptionTransitionCurlDown
	UIViewAnimationOptionTransitionCrossDissolve 
	UIViewAnimationOptionTransitionFlipFromTop 
	UIViewAnimationOptionTransitionFlipFromBottom

除了UIViewAnimationOptionTransitionCrossDissolve之外，剩下的值和CATransition类型完全没关系。

小例子 使用UIKit提供的方法来做过渡动画

	@interface ViewController ()
	@property (nonatomic, weak) IBOutlet UIImageView *imageView;
	@property (nonatomic, copy) NSArray *images;
	@end
	@implementation ViewController
	- (void)viewDidLoad
	{
    	[super viewDidLoad]; // 设置图片
    	self.images = @[[UIImage imageNamed:@"Anchor.png"],
          	        	[UIImage imageNamed:@"Cone.png"],
            	       	[UIImage imageNamed:@"Igloo.png"],
                	    [UIImage imageNamed:@"Spaceship.png"]];
	- (IBAction)switchImage
	{
    	[UIView transitionWithView:self.imageView duration:1.0
        	               options:UIViewAnimationOptionTransitionFlipFromLeft
            	        animations:^{
                	        // 切换到下一张图片
                   		     UIImage *currentImage = self.imageView.image;
                   	    	 NSUInteger index = [self.images indexOfObject:currentImage];
                   		     index = (index + 1) % [self.images count];
                   	    	 self.imageView.image = self.images[index];
                    	}
                    	completion:NULL];
	}

	@end


过渡动画的原则就是对原始的图层外观截图，然后添加一段动画，平滑过渡到图层改变之后那个截图的效果。

如果我们知道如何对图层截图，我们就可以使用属性动画来代替CATransition或者是UIKit的过渡方法来实现动画。

CALayer有一个-renderInContext:方法，可以通过把它绘制到Core Graphics的上下文中捕获当前内容的图片，然后在另外的视图中显示出来。如果我们把这个截屏视图置于原始视图之上，就可以遮住真实视图的所有变化，于是重新创建了一个简单的过渡效果。

小例子：用renderInContext:创建自定义过渡效果

	@implementation ViewController
	- (IBAction)performTransition
	{
    	// 截图
    	UIGraphicsBeginImageContextWithOptions(self.view.bounds.size, YES, 0.0);
    	[self.view.layer renderInContext:UIGraphicsGetCurrentContext()];
    	UIImage *coverImage = UIGraphicsGetImageFromCurrentImageContext();
    	// 插入截图
    	UIView *coverView = [[UIImageView alloc] initWithImage:coverImage];
    	coverView.frame = self.view.bounds;
    	[self.view addSubview:coverView];
    	// 更新视图（设置一个随机的背景色）
    	CGFloat red = arc4random() / (CGFloat)INT_MAX;
    	CGFloat green = arc4random() / (CGFloat)INT_MAX;
    	CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    	self.view.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
    	// 做自定义动画
    	[UIView animateWithDuration:1.0 animations:^{
        	// 放缩旋转和淡入淡出
        	CGAffineTransform transform = CGAffineTransformMakeScale(0.01, 0.01);
        	transform = CGAffineTransformRotate(transform, M_PI_2);
        	coverView.transform = transform;
        	coverView.alpha = 0.0;
    	} completion:^(BOOL finished) {
        	// 动画结束移除覆盖层
        	[coverView removeFromSuperview];
    	}];
	}
	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced8.5.png)

###8.4、取消进行中的动画

之前说过，可以用-addAnimation:forKey:方法中的key参数来在添加动画之后检索一个动画

	- (CAAnimation *)animationForKey:(NSString *)key;

但并不支持在动画运行过程中修改动画，所以这个方法主要用来检测动画的属性，或者判断它是否被添加到当前图层中。

为了终止一个指定的动画，你可以用如下方法把它从图层移除掉：

	- (void)removeAnimationForKey:(NSString *)key;

或者移除所有动画：

	- (void)removeAllAnimations;

动画一旦被移除，图层的外观就`立刻`更新到当前的模型图层的值。

一般说来，动画在结束之后被自动移除，除非设置removedOnCompletion为NO，如果你设置动画在结束之后不被自动移除，那么当它不需要的时候你要手动移除它；否则它会一直存在于内存中，直到图层被销毁。

小例子：开始和停止一个动画，用之前旋转飞船的代码，增加一个按钮来停止或启动动画，-animationDidStop:finished:方法的flag参数表面动画是否停止

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;
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

	- (IBAction)start
	{
    	// 做旋转动画
    	CABasicAnimation *animation = [CABasicAnimation animation];
    	animation.keyPath = @"transform.rotation";
    	animation.duration = 2.0;
    	animation.byValue = @(M_PI * 2);
    	animation.delegate = self;
    	[self.shipLayer addAnimation:animation forKey:@"rotateAnimation"];
	}

	- (IBAction)stop
	{
    	[self.shipLayer removeAnimationForKey:@"rotateAnimation"];
	}

	- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag
	{
    	// 打印动画是否停止
    	NSLog(@"The animation stopped (finished: %@)", flag? @"YES": @"NO");
	}

	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced8.6.png)
