---
layout: post
title: "Core Animation Advanced Technique 学习笔记(6)"
date: 2014-08-31 16:19:52 +0800
comments: true
categories: iOS-CoreAnimation
---

#第二部分：运动中的设置
##7、隐式动画（Implicit Animations）
###7.1、事务

Core Animation是基于一个说屏幕上的任何东西都可能做动画的假设为前提的。Core Animation中动画的默认开启，要关闭需要手动设置。

当改变CALayer那个是否可以做动画的属性时，并不能立刻在屏幕上体现出来。相反，它是从先前的值平滑过渡到新的值。这一切都是默认的行为，你不需要做额外的操作。

小例子：创建一个蓝色方块，然后添加一个按钮，随机改变图层颜色

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
    	// 添加到视图尚
    	[self.layerView.layer addSublayer:self.colorLayer];
	}

	- (IBAction)changeColor
	{
    	// 随机给背景配色
    	CGFloat red = arc4random() / (CGFloat)INT_MAX;
    	CGFloat green = arc4random() / (CGFloat)INT_MAX;
    	CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    	self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;                                                                                       ￼
	}

	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced7.1.jpg)

这其实就是所谓的隐式动画。之所以叫内联是因为我们并没有指定任何动画的类型。改变了一个属性，然后Core Animation来决定如何并且何时去做动画。Core Animaiton同样支持显式动画，之后会说明。

当你改变一个属性，Core Animation是如何判断动画类型和持续时间的呢？实际上动画执行的时间取决于当前事务的设置，动画类型取决于图层行为。

事务指的是Core Animation用来包含一系列属性动画集合的机制，任何用指定事务去改变做动画的图层，属性都不会立刻发生变化，而是当事务一旦提交的时候开始用一个动画过渡到新值。

事务是通过CATransaction类来做管理，这个类不是去管理一个简单的事务，而是管理了很多不能访问的事务。

CATransaction没有属性或者实例方法，并且也不能用+alloc和-init方法创建它。但是可以用+begin和+commit分别来入栈或者出栈。

任何可以做动画的图层属性都会被添加到栈顶的事务，你可以通过+setAnimationDuration:方法设置当前事务的动画时间，或者通过+animationDuration方法来获取值（默认0.25秒）。

Core Animation在每个run loop周期中自动开始一次新的事务（run loop是iOS负责收集用户输入，处理定时器或者网络事件并且重新绘制屏幕的东西），即使你不显式的用[CATransaction begin]开始一次事务，任何在一次run loop循环中属性的改变都会被集中起来，然后做一次0.25秒的动画。

明白这些之后，我们就可以轻松修改改变颜色的动画的时间了。我们当然可以用当前事务的+setAnimationDuration:方法来修改动画时间，但在这里我们首先开始一个新的事务，于是修改时间就不会有别的副作用。

因为修改当前事务的时间可能会导致同一时刻别的动画（如屏幕旋转），所以最好还是在调整动画之前压入一个新的事务。

修改后的例子：使用CATransaction控制动画时间

	- (IBAction)changeColor
	{
    	// 开始一个新的事务
    	[CATransaction begin];
    	// 设置1秒的动画
    	[CATransaction setAnimationDuration:1.0];
    	// 随机给背景配色
    	CGFloat red = arc4random() / (CGFloat)INT_MAX;
    	CGFloat green = arc4random() / (CGFloat)INT_MAX;
    	CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    	self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;
    ￼	// 提交事务
    	[CATransaction commit];
	}
	
如果你用过UIView的动画方法，那么应该对这个模式不陌生。

UIView有两个方法，+beginAnimations:context:和+commitAnimations，和CATransaction的+begin和+commit方法类似。实际上在+beginAnimations:context:和+commitAnimations之间所有视图或者图层属性的改变而做的动画都是由于设置了CATransaction的原因。

在iOS4中，苹果对UIView添加了一种基于block的动画方法：+animateWithDuration:animations:。这样写对做一堆的属性动画在语法上会更加简单，但实质上它们都是在做同样的事情。

CATransaction的+begin和+commit方法在+animateWithDuration:animations:内部自动调用，这样block中所有属性的改变都会被事务所包含。这样也可以避免开发者由于对+begin和+commit匹配的失误造成的风险。

###7.2、完成块

基于UIView的block的动画允许你在动画结束的时候提供一个完成的动作。

CATranscation接口提供的+setCompletionBlock:方法也有同样的功能。

小例子 在颜色动画完成之后添加一个回调

	- (IBAction)changeColor
	{
    	// 开始新的事务
    	[CATransaction begin];
    	// 设置动画时间1秒
    	[CATransaction setAnimationDuration:1.0];
    	// 添加动画完成的操作
    	[CATransaction setCompletionBlock:^{
        	// 旋转图层90度
        	CGAffineTransform transform = self.colorLayer.affineTransform;
        	transform = CGAffineTransformRotate(transform, M_PI_2);
        	self.colorLayer.affineTransform = transform;
    	}];
    	// 随机给背景配色
    	CGFloat red = arc4random() / (CGFloat)INT_MAX;
    	CGFloat green = arc4random() / (CGFloat)INT_MAX;
    	CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    	self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;
    	// 提交事务
    	[CATransaction commit];
	}

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced7.2.jpg)

注意旋转动画要比颜色渐变快得多，这是因为完成块是在颜色渐变的事务提交并出栈之后才被执行，于是，用默认的事务做变换，默认的时间也就变成了0.25秒。


###7.3、图层动作

现在来做个实验，试着直接对UIView关联的图层做动画而不是一个单独的图层。

小例子：直接设置图层的属性

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *layerView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 设置layerView寄宿图层的背景色
    	self.layerView.layer.backgroundColor = [UIColor blueColor].CGColor;
	}

	- (IBAction)changeColor
	{
    	// 开始一个新事务
    	[CATransaction begin];
    	// 设置一秒动画
    	[CATransaction setAnimationDuration:1.0];
    	// 随机给图层背景配色
    	CGFloat red = arc4random() / (CGFloat)INT_MAX;
    	CGFloat green = arc4random() / (CGFloat)INT_MAX;
    	CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    	self.layerView.layer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;
    	// 提交事务
    	[CATransaction commit];
	}
	
运行程序，你会发现当按下按钮，图层颜色瞬间切换到新的值，而不是之前平滑过渡的动画。

发生了什么呢？隐式动画好像被UIView关联图层给禁用了。

首先，需要知道隐式动画的实现方式。

改变属性时CALayer自动应用的动画就是行为，当CALayer的属性被修改时候，它会调用-actionForKey:方法，传递属性的名称。剩下的操作都在CALayer的头文件中有详细的说明，实质上是如下几步：

- 图层首先检测它是否有委托，并且是否实现CALayerDelegate协议指定的-actionForLayer:forKey方法。如果有，直接调用并返回结果。

- 如果没有委托，或者委托没有实现-actionForLayer:forKey方法，图层接着检查包含属性名称对应行为映射的actions字典。

- 如果actions字典没有包含对应的属性，那么图层接着在它的style字典接着搜索属性名。

- 最后，如果在style里面也找不到对应的行为，那么图层将会直接调用定义了每个属性的标准行为的-defaultActionForKey:方法。

- 所以一轮完整的搜索结束之后，-actionForKey:要么返回空（这种情况下将不会有动画发生），要么是CAAction协议对应的对象，最后CALayer拿这个结果去对先前和当前的值做动画。

于是这就解释了UIKit是如何禁用隐式动画的：

	每个UIView对它关联的图层都扮演了一个委托，并且提供了-actionForLayer:forKey的实现方法。当不在一个动画块的实现中，UIView对所有图层行为返回nil，但是在动画block范围之内，它就返回了一个非空值。

小例子：测试UIView的actionForLayer:forKey:实现

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *layerView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 测试图层动作
    	NSLog(@"Outside: %@", [self.layerView actionForLayer:self.layerView.layer forKey:@"backgroundColor"]);
    	// 开始动画块
    	[UIView beginAnimations:nil context:nil];
    	// 测试图层动作
    	NSLog(@"Inside: %@", [self.layerView actionForLayer:self.layerView.layer forKey:@"backgroundColor"]);
    	// 结束动画块
    	[UIView commitAnimations];
	}

	@end
	
运行程序，控制台显示结果如下：

	$ LayerTest[21215:c07] Outside: <null>
	$ LayerTest[21215:c07] Inside: <CABasicAnimation: 0x757f090>

于是可以预言，当属性在动画块之外发生改变，UIView直接通过返回nil来禁用隐式动画。

但如果在动画块范围之内，根据动画具体类型返回相应的属性。

当然返回nil并不是禁用隐式动画唯一的办法，CATransacition有个方法叫做+setDisableActions:，可以用来对所有属性打开或者关闭隐式动画。

如果在之前的[CATransaction begin]之后添加下面的代码，同样也会阻止动画的发生：

	[CATransaction setDisableActions:YES];

总结一下，我们知道了如下几点

- UIView关联的图层禁用了隐式动画，对这种图层做动画的唯一办法就是使用UIView的动画函数（而不是依赖CATransaction），或者继承UIView，并覆盖-actionForLayer:forKey:方法，或者直接创建一个显式动画。

- 对于单独存在的图层，我们可以通过实现图层的-actionForLayer:forKey:委托方法，或者提供一个actions字典来控制隐式动画。

我们来对颜色渐变的例子使用一个不同的行为，要么可以通过给colorLayer设置一个自定义的actions字典，或者也可以使用委托来实现，但是actions字典可以写更少的代码。

到底如何创建一个合适的行为对象呢？

行为通常是一个被Core Animation隐式调用的显式动画对象。

这里我们使用的是一个实现了CATransaction的实例，叫做推进过渡。CATransition能响应CAAction协议，并且可以当做一个图层行为

小例子：实现自定义行为

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
    	// 增加自定义行为
    	CATransition *transition = [CATransition animation];
    	transition.type = kCATransitionPush;
    	transition.subtype = kCATransitionFromLeft;
    	self.colorLayer.actions = @{@"backgroundColor": transition};
    	// 添加到视图尚
    	[self.layerView.layer addSublayer:self.colorLayer];
	}

	- (IBAction)changeColor
	{
    	// 随机给图层背景配色
    	CGFloat red = arc4random() / (CGFloat)INT_MAX;
    	CGFloat green = arc4random() / (CGFloat)INT_MAX;
    	CGFloat blue = arc4random() / (CGFloat)INT_MAX;
    	self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;
	}

	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced7.3.jpg)

###7.4、表现与模型

CALayer的属性行为,在改变一个图层的属性后并没有立刻生效，而是通过一段时间渐变更新。这是怎么做到的呢？

当你改变一个图层的属性，属性值的确是立刻更新的（如果你读取它的数据，你会发现它的值在你设置它的那一刻就已经生效了），但是屏幕上并没有马上发生改变。

这是因为你设置的属性并没有直接调整图层的外观，相反，他只是定义了图层动画结束之后将要变化的外观。

当设置CALayer的属性，实际上是在定义当前事务结束之后图层如何显示的模型。

Core Animation扮演了一个控制器的角色，并且负责根据图层行为和事务设置去不断更新视图的这些属性在屏幕上的状态。

CALayer是一个连接用户界面（就是MVC中的view）虚构的类，但是在界面本身这个场景下，CALayer的行为更像是存储了视图如何显示和动画的数据模型。

实际上，在苹果自己的文档中，图层树通常都是值的图层树模型。

在iOS中，屏幕每秒钟重绘60次。如果动画时长比60分之一秒要长，Core Animation就需要在设置一次新值和新值生效之间，对屏幕上的图层进行重新组织。这意味着CALayer除了“真实”值（就是你设置的值）之外，必须要知道当前显示在屏幕上的属性值的记录。

每个图层属性的显示值都被存储在一个叫做表现图层的独立图层当中，他可以通过-presentationLayer方法来访问。

这个图层实际上是模型图层的复制，但是它的属性值代表了在任何指定时刻当前外观效果。

换句话说，你可以通过表现图层的值来获取当前屏幕上真正显示出来的值（图7.4）。

表现树通过图层树中所有图层的表现图层所形成。注：表现图层仅仅当图层首次被提交（就是首次第一次在屏幕上显示）的时候创建，所以在那之前调用-presentationLayer将会返回nil。

你可能注意到有一个叫做–modelLayer的方法。在表现图层上调用–modelLayer将会返回它正在呈现所依赖的CALayer。通常在一个图层上调用-modelLayer会返回–self（实际上我们已经创建的原始图层就是一种数据模型）。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced7.4.jpg)

大多数情况下，你不需要直接访问表现图层，你可以通过和模型图层的交互，来让Core Animation更新显示。两种情况下呈现图层会变得很有用，一个是同步动画，一个是处理用户交互。

如果你在实现一个基于定时器的动画，而不仅仅是基于事务的动画，这个时候准确地知道在某一时刻图层显示在什么位置就会对正确摆放图层很有用了。

如果你想让你做动画的图层响应用户输入，你可以使用-hitTest:方法来判断指定图层是否被触摸，这时候对表现图层而不是模型图层调用-hitTest:会显得更有意义，因为表现图层代表了用户当前看到的图层位置，而不是当前动画结束之后的位置。

小例子：使用presentationLayer图层来判断当前图层位置（点击屏幕的任意位置会让图层平移到哪里，点击图层本身则会随机改变它的颜色，通过对表现图层调用-hitTest:来判断是否被点击，如果让-hitTest:直接作用于colorLayer而不是表现图层，则图层移动时不能正确显示，这时候就需要点击图层将要移动到的那个位置而不是图层本身来响应点击）

	@interface ViewController ()

	@property (nonatomic, strong) CALayer *colorLayer;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建一个红色图层
	    self.colorLayer = [CALayer layer];
   	 	self.colorLayer.frame = CGRectMake(0, 0, 100, 100);
    	self.colorLayer.position = CGPointMake(self.view.bounds.size.width / 2, self.view.bounds.size.height / 2);
    	self.colorLayer.backgroundColor = [UIColor redColor].CGColor;
    	[self.view.layer addSublayer:self.colorLayer];
	}

	- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
	{
    	// 得到触摸点
	    CGPoint point = [[touches anyObject] locationInView:self.view];
    	// 检查是否点击了移动中的图层
    	if ([self.colorLayer.presentationLayer hitTest:point]) {
	        // 给图层背景随机配色
    	    CGFloat red = arc4random() / (CGFloat)INT_MAX;
        	CGFloat green = arc4random() / (CGFloat)INT_MAX;
       	 	CGFloat blue = arc4random() / (CGFloat)INT_MAX;
        	self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;
    	} else {
        	// 否则移动图层到新的位置
        	[CATransaction begin];
        	[CATransaction setAnimationDuration:4.0];
        	self.colorLayer.position = point;
        	[CATransaction commit];
    	}
	}
	@end