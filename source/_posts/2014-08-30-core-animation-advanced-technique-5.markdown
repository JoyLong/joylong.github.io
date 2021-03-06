---
layout: post
title: "Core Animation Advanced Technique 学习笔记(5)"
date: 2014-08-30 18:52:47 +0800
comments: true
categories: iOS-CoreAnimation
---

#第一部分：下面的图层
##6、专用图层(Specialized Layers)

###6.1、CAShapeLayer

CAShapeLayer是一个通过矢量图形而不是bitmap来绘制的图层子类。

你指定诸如颜色和线宽等属性，用CGPath来定义想要绘制的图形，最后CAShapeLayer就会自动渲染出来。

当然，也可以用Core Graphics直接向原始的CALyer的内容中绘制路径，不过，使用CAShapeLayer有以下一些优点：

- 快。CAShapeLayer使用了硬件加速，绘制比Core Graphics快很多。

- 高效使用内存。CAShapeLayer不需要像普通CALayer一样创建一个寄宿图形，所以无论有多大，都不会占用太多的内存。

- 不会被图层边界剪裁掉。CAShapeLayer可以在边界之外绘制。图层路径不会像在使用Core Graphics的CALayer一样被剪裁掉。

- 不会出现像素化。当你给CAShapeLayer做3D变换时，它不像一个有寄宿图的普通图层一样变得像素化。

####6.1.1、创建一个CGPath

CAShapeLayer可以用来绘制所有能够通过CGPath来表示的形状。

形状不一定要闭合，图层路径也不一定要不可破，你可以在一个图层上绘制好几个不同的形状。可以控制一些属性比如lineWith（线宽，用点表示单位），lineCap（线条结尾的样子），和lineJoin（线条之间的结合点的样子）；

但是在图层层面你只有一次机会设置这些属性。

如果你想用不同颜色或风格来绘制多个形状，就不得不为每个形状准备一个图层了。

下面的代码用一个CAShapeLayer绘制一个简单的火柴人。
CAShapeLayer属性是CGPathRef类型，但是我们用UIBezierPath帮助类创建了图层路径，这样我们就不用考虑人工释放CGPath了。

	#import "DrawingView.h"
	#import <QuartzCore/QuartzCore.h>

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
	  	[super viewDidLoad];
  		// 创建路径
  		UIBezierPath *path = [[UIBezierPath alloc] init];
  		[path moveToPoint:CGPointMake(175, 100)];
	  	[path addArcWithCenter:CGPointMake(150, 100) radius:25 startAngle:0 endAngle:2*M_PI clockwise:YES];
 	 	[path moveToPoint:CGPointMake(150, 125)];
  		[path addLineToPoint:CGPointMake(150, 175)];
  		[path addLineToPoint:CGPointMake(125, 225)];
  		[path moveToPoint:CGPointMake(150, 175)];
  		[path addLineToPoint:CGPointMake(175, 225)];
  		[path moveToPoint:CGPointMake(100, 150)];
  		[path addLineToPoint:CGPointMake(200, 150)];
  		
		// 创建CAShapeLayer
  		CAShapeLayer *shapeLayer = [CAShapeLayer layer];
  		shapeLayer.strokeColor = [UIColor redColor].CGColor;
  		shapeLayer.fillColor = [UIColor clearColor].CGColor;
  		shapeLayer.lineWidth = 5;
  		shapeLayer.lineJoin = kCALineJoinRound;
  		shapeLayer.lineCap = kCALineCapRound;
  		shapeLayer.path = path.CGPath;
  		// 添加到视图上
  		[self.containerView.layer addSublayer:shapeLayer];
	}
	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.1.png)

####6.1.2、用CAShapeLayer为每个角指定是否圆角

之前提到了CAShapeLayer为创建圆角视图提供了一个方法，就是CALayer的cornerRadius属性。

虽然使用CAShapeLayer类需要更多的工作，但是它有一个优势就是可以单独指定每个角。

下面这段代码绘制了一个有三个圆角一个直角的矩形：

	// 定义路径参数
	CGRect rect = CGRectMake(50, 50, 100, 100);
	CGSize radii = CGSizeMake(20, 20);
	UIRectCorner corners = UIRectCornerTopRight | UIRectCornerBottomRight | UIRectCornerBottomLeft;
	// 创建路径
	UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:rect byRoundingCorners:corners cornerRadii:radii];

我们可以通过这个图层路径绘制一个既有直角又有圆角的视图。

如果我们想依照此图形来剪裁视图内容，我们可以把CAShapeLayer作为视图的宿主图层(backing layer)，而不是添加一个子视图。

###6.2、CATextLayer

如果你想在一个图层里面显示文字，完全可以借助图层代理直接将字符串使用Core Graphics写入图层的内容（这就是UILabel的精髓）。如果越过寄宿于图层的视图，直接在图层上操作，那其实相当繁琐。你要为每一个显示文字的图层创建一个能像图层代理一样工作的类，还要逻辑上判断哪个图层需要显示哪个字符串，更别提还要记录不同的字体，颜色等一系列乱七八糟的东西。

幸运的是，Core Animation提供了一个CALayer的子类CATextLayer，它以图层的形式包含了UILabel几乎所有的绘制特性，并且额外提供了一些新的特性。

同样，CATextLayer也要比UILabel渲染得快得多。

iOS 6及之前的版本，UILabel其实是通过WebKit来实现绘制的，这样就造成了当有很多文字的时候就会有极大的性能压力。

而CATextLayer使用了Core text，并且渲染得非常快。

####6.2.1、用CATextLayer来实现一个UILabel

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *labelView;

	@end

	@implementation ViewController
	- (void)viewDidLoad
	{
  		[super viewDidLoad];

  		// 创建一个CATextLayer
  		CATextLayer *textLayer = [CATextLayer layer];
  		textLayer.frame = self.labelView.bounds;
  		[self.labelView.layer addSublayer:textLayer];

  		// 设置文本属性
  		textLayer.foregroundColor = [UIColor blackColor].CGColor;
  		textLayer.alignmentMode = kCAAlignmentJustified;
  		textLayer.wrapped = YES;

  		// 选择字体
  		UIFont *font = [UIFont systemFontOfSize:15];

  		// 设置图层字体
  		CFStringRef fontName = (__bridge CFStringRef)font.fontName;
  		CGFontRef fontRef = CGFontCreateWithFontName(fontName);
  		textLayer.font = fontRef;
  		textLayer.fontSize = font.pointSize;
  		CGFontRelease(fontRef);

  		// 设置文本
  		NSString *text = @"Lorem ipsum dolor sit amet, consectetur adipiscing \ elit. Quisque massa arcu, eleifend vel varius in, facilisis pulvinar \ leo. Nunc quis nunc at mauris pharetra condimentum ut ac neque. Nunc elementum, libero ut porttitor dictum, diam odio congue lacus, vel \ fringilla sapien diam at purus. Etiam suscipit pretium nunc sit amet \ lobortis";

  		// 设置图层文本
  		textLayer.string = text;
	}
	@end
	
![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.2.png)


如果你放大看这个文本，会发现这些文本有一些像素化了。

这是因为并没有以Retina的方式渲染，之前提到过contentScale属性，是用来决定图层内容应该以怎样的分辨率来渲染。

contentsScale并不关心屏幕的拉伸因素而总是默认为1.0。如果我们想以Retina的质量来显示文字，我们就得手动地设置CATextLayer的contentsScale属性，

如下：

	textLayer.contentsScale = [UIScreen mainScreen].scale;

这样就解决了这个问题，如下图

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.3.png)

CATextLayer的font属性并不是一个UIFont，而是一个CFTypeRef类型。

这样可以根据你的具体需要来决定字体属性应该是用CGFontRef类型还是CTFontRef类型（Core Text字体）。

同时字体大小也是用fontSize属性单独设置的，因为CTFontRef和CGFontRef并不像UIFont一样包含点大小。

这个例子会告诉你如何将UIFont转换成CGFontRef。

另外，CATextLayer的string属性并不是你想象的NSString类型，而是id类型。这样你既可以用NSString也可以用NSAttributedString来设置文本了（注：NSAttributedString并不是NSString的子类）。

####6.2.2、富文本

iOS 6中，Apple给UILabel和其他UIKit文本视图添加了对属性化字符串（NSAttributedString）的支持，应该说这是一个很方便的特性。

但是从iOS3.2开始CATextLayer就已经支持属性化字符串了。这样的话，如果你想要支持更低版本的iOS系统，CATextLayer无疑是你向界面中增加富文本的好办法，而且也不用去跟复杂的Core Text打交道，也省了用UIWebView的麻烦。

用NSAttributedString实现一个富文本标签。

	#import "DrawingView.h"
	#import <QuartzCore/QuartzCore.h>
	#import <CoreText/CoreText.h>

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *labelView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
  		[super viewDidLoad];

  		// 创建一个文本图层
  		CATextLayer *textLayer = [CATextLayer layer];
  		textLayer.frame = self.labelView.bounds;
  		textLayer.contentsScale = [UIScreen mainScreen].scale;
  		[self.labelView.layer addSublayer:textLayer];

	  	// 设置文本属性
  		textLayer.alignmentMode = kCAAlignmentJustified;
  		textLayer.wrapped = YES;

  		// 设置字体
  		UIFont *font = [UIFont systemFontOfSize:15];

  		// 设置文本
   		NSString *text = @"Lorem ipsum dolor sit amet, consectetur adipiscing \ elit. Quisque massa arcu, eleifend vel varius in, facilisis pulvinar \ leo. Nunc quis nunc at mauris pharetra condimentum ut ac neque. Nunc \ elementum, libero ut porttitor dictum, diam odio congue lacus, vel \ fringilla sapien diam at purus. Etiam suscipit pretium nunc sit amet \ lobortis"; ￼
  		// 创建属性字符串
  		NSMutableAttributedString *string = nil;
  		string = [[NSMutableAttributedString alloc] initWithString:text];

  		// 把UIFont转换成CTFont
  		CFStringRef fontName = (__bridge CFStringRef)font.fontName;
  		CGFloat fontSize = font.pointSize;
  		CTFontRef fontRef = CTFontCreateWithName(fontName, fontSize, NULL);

  		// 设置文本属性
  		NSDictionary *attribs = @{
    		(__bridge id)kCTForegroundColorAttributeName:(__bridge id)[UIColor blackColor].CGColor,
    		(__bridge id)kCTFontAttributeName: (__bridge id)fontRef
  		};
  		
  		[string setAttributes:attribs range:NSMakeRange(0, [text length])];
  			attribs = @{
    		(__bridge id)kCTForegroundColorAttributeName: (__bridge id)[UIColor redColor].CGColor,
    		(__bridge id)kCTUnderlineStyleAttributeName: @(kCTUnderlineStyleSingle),
    		(__bridge id)kCTFontAttributeName: (__bridge id)fontRef
  		};
  		[string setAttributes:attribs range:NSMakeRange(6, 5)];

  		// 释放CTFont
  		CFRelease(fontRef);

  		// 设置图层文本
  		textLayer.string = string;
	}
	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.4.png)

####6.2.3、行距和字距

由于绘制的实现机制不同（Core Text和WebKit），用CATextLayer渲染和用UILabel渲染出的文本行距和字距也不是不尽相同的。二者的差异程度（由使用的字体和字符决定）总的来说挺小。

但是如果你想正确的显示普通便签和CATextLayer就一定要记住这一点。

已经可以确定的是CATextLayer比UILabel有着更好的性能表现，同时还有额外的布局选项并且在iOS 5上支持富文本。

但是与一般的标签比较而言会更加繁琐一些。如果我们真的在需求一个UILabel的可用替代品，应该继承UILabel，然后添加一个子图层CATextLayer并重写显示文本的方法。

但是仍然会有由UILabel的-drawRect:方法创建的空寄宿图。而且由于CALayer不支持自动缩放和自动布局，子视图并不是主动跟踪视图边界的大小，所以每次视图大小被更改，我们不得不手动更新子图层的边界。

我们真正想要的是一个用CATextLayer作为宿主图层的UILabel子类，这样就可以随着视图自动调整大小而且也没有冗余的寄宿图啦。这个图层是由视图自动创建和管理的，一旦被创建，我们就无法替代这个图层了。但是如果继承了UIView，那我们就可以重写+layerClass方法使得在创建的时候能返回一个不同的图层子类。UIView会在初始化的时候调用+layerClass方法，然后用它的返回类型来创建宿主图层。

下面的例子演示了使用CATextLayer的UILabel子类：LayerLabel，而不是调用一般的UILabel使用的较慢的-drawRect：方法。

	#import "LayerLabel.h"
	#import <QuartzCore/QuartzCore.h>

	@implementation LayerLabel
	+ (Class)layerClass
	{
  		// 这个方法让Label创建一个CATextLayer而不是原本的CALayer作为寄宿图
  		return [CATextLayer class];
	}

	- (CATextLayer *)textLayer
	{
  		return (CATextLayer *)self.layer;
	}

	- (void)setUp
	{
  		// 根据UILabel的设置来设置默认项
  		self.text = self.text;
  		self.textColor = self.textColor;
  		self.font = self.font;


  		
  		[self textLayer].alignmentMode = kCAAlignmentJustified;
  ￼
  		[self textLayer].wrapped = YES;
  		[self.layer display];
	}

	- (id)initWithFrame:(CGRect)frame
	{
  		// 创建Label时自动调用
  		if (self = [super initWithFrame:frame]) {
    		[self setUp];
  		}
  		return self;
	}

	- (void)awakeFromNib
	{
  		// 使用IB创建时调用
  		[self setUp];
	}

	- (void)setText:(NSString *)text
	{
  		super.text = text;
  		// 设置图层颜色
  		[self textLayer].string = text;
	}

	- (void)setTextColor:(UIColor *)textColor
	{
  		super.textColor = textColor;
  		// 设置图层文本颜色
  		[self textLayer].foregroundColor = textColor.CGColor;
	}

	- (void)setFont:(UIFont *)font
	{
  		super.font = font;
  		// 设置图层字体
  		CFStringRef fontName = (__bridge CFStringRef)font.fontName;
  		CGFontRef fontRef = CGFontCreateWithFontName(fontName);
  		[self textLayer].font = fontRef;
  		[self textLayer].fontSize = font.pointSize;
  ￼	
  		CGFontRelease(fontRef);
	}
	@end
	
如果你运行代码，你会发现文本并没有模糊，而我们也没有设置contentsScale属性。把CATextLayer作为宿主图层的另一好处就是视图自动设置了contentsScale属性。

如果你打算支持iOS 6及以上，基于CATextLayer的标签可能就有有些局限。

但是总得来说，如果想在app里面充分利用CALayer子类，用+layerClass来创建基于不同图层的视图是一个简单可复用的方法。


###6.3、CATransformLayer

Core Animation图层很容易就可以让你在2D环境下做出层级体系下的变换，但是3D情况下就不太可能，因为所有的图层都把他的子类都平面化到一个场景中。

CATransformLayer解决了这个问题，CATransformLayer不同于普通的CALayer，因为它不能显示它自己的内容。存在的前提是：只有当存在了一个能作用于子图层的变换它才真正存在。

CATransformLayer并不平面化它的子图层，所以它能够用于构造一个层级的3D结构。

之前讲过，我们将通过旋转camera来解决图层平面化问题,而不是像立方体示例代码中用的sublayerTransform。这是一个非常不错的技巧，但是只能作用域单个对象上，如果你的场景包含两个立方体，那我们就不能用这个技巧单独旋转他们了。

如果用CATransformLayer，第一个问题就来了：之前我们是用多个视图来构造了我们的立方体，而不是单独的图层。我们不能在不打乱已有的视图层次的前提下在一个本身不是有寄宿图的图层中放置一个寄宿图图层。我们可以创建一个新的UIView子类寄宿在CATransformLayer（用+layerClass方法）之上。但是，为了简化案例，我们仅仅重建了一个单独的图层，而不是使用视图。这意味着我们不能像之前一样在立方体表面显示按钮和标签，不过我们现在也用不到这个特性。

下面代码就是用之前的逻辑放置立方体。但是并不像以前那样直接将立方面添加到容器视图的宿主图层，我们将他们放置到一个CATransformLayer中创建一个独立的立方体对象，然后将两个这样的立方体放进容器中。我们随机地给立方面染色以将他们区分开来，这样就不用靠标签或是高亮来区分他们。

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;

	@end

	@implementation ViewController

	- (CALayer *)faceWithTransform:(CATransform3D)transform
	{
  		// 创建立方体面的图层
  		CALayer *face = [CALayer layer];
  		face.frame = CGRectMake(-50, -50, 100, 100);

  		// 选择随机色
  		CGFloat red = (rand() / (double)INT_MAX);
  		CGFloat green = (rand() / (double)INT_MAX);
  		CGFloat blue = (rand() / (double)INT_MAX);
  		face.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;

  		￼// 设置transform并返回
  		face.transform = transform;
  		return face;
	}

	- (CALayer *)cubeWithTransform:(CATransform3D)transform
	{
  		// 创建立方体图层
  		CATransformLayer *cube = [CATransformLayer layer];

  		// 添加立方体面1
  		CATransform3D ct = CATransform3DMakeTranslation(0, 0, 50);
  		[cube addSublayer:[self faceWithTransform:ct]];

  		// 添加立方体面2
  		ct = CATransform3DMakeTranslation(50, 0, 0);
  		ct = CATransform3DRotate(ct, M_PI_2, 0, 1, 0);
  		[cube addSublayer:[self faceWithTransform:ct]];

  		// 添加立方体面3
  		ct = CATransform3DMakeTranslation(0, -50, 0);
  		ct = CATransform3DRotate(ct, M_PI_2, 1, 0, 0);
  		[cube addSublayer:[self faceWithTransform:ct]];

  		// 添加立方体面4
  		ct = CATransform3DMakeTranslation(0, 50, 0);
  		ct = CATransform3DRotate(ct, -M_PI_2, 1, 0, 0);
  		[cube addSublayer:[self faceWithTransform:ct]];

  		// 添加立方体面5
  		ct = CATransform3DMakeTranslation(-50, 0, 0);
  		ct = CATransform3DRotate(ct, -M_PI_2, 0, 1, 0);
  		[cube addSublayer:[self faceWithTransform:ct]];

  		// 添加立方体面6
  		ct = CATransform3DMakeTranslation(0, 0, -50);
  		ct = CATransform3DRotate(ct, M_PI, 0, 1, 0);
  		[cube addSublayer:[self faceWithTransform:ct]];

  		// 让立方体图层居中
  		CGSize containerSize = self.containerView.bounds.size;
  		cube.position = CGPointMake(containerSize.width / 2.0, containerSize.height / 2.0);

  		// 设置transform并返回
  		cube.transform = transform;
  		return cube;
	}

	- (void)viewDidLoad
	{￼
  		[super viewDidLoad];

  		// 设定透视变换
  		CATransform3D pt = CATransform3DIdentity;
  		pt.m34 = -1.0 / 500.0;
  		self.containerView.layer.sublayerTransform = pt;

  		// 设定立方体1的透视并添加在图层上
  		CATransform3D c1t = CATransform3DIdentity;
  		c1t = CATransform3DTranslate(c1t, -100, 0, 0);
  		CALayer *cube1 = [self cubeWithTransform:c1t];
  		[self.containerView.layer addSublayer:cube1];

  		// 设定立方体2的透视并添加在图层上
  		CATransform3D c2t = CATransform3DIdentity;
  		c2t = CATransform3DTranslate(c2t, 100, 0, 0);
  		c2t = CATransform3DRotate(c2t, -M_PI_4, 1, 0, 0);
  		c2t = CATransform3DRotate(c2t, -M_PI_4, 0, 1, 0);
  		CALayer *cube2 = [self cubeWithTransform:c2t];
  		[self.containerView.layer addSublayer:cube2];
	}
	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.5.png)

###6.4、CAGradientLayer

CAGradientLayer是用来生成两种或更多颜色平滑渐变的。

用Core Graphics复制一个CAGradientLayer并将内容绘制到一个普通图层的寄宿图也可以实现这个效果，但是CAGradientLayer的真正好处在于绘制使用了硬件加速。

####6.4.1、基础渐变

我们将从一个简单的红变蓝的对角线渐变开始.这些渐变色彩放在一个数组中，并赋给colors属性。

但是这个数组成员接受CGColorRef类型的值（并不是从NSObject派生而来），因此需要通过bridge转换以确保编译正常。

CAGradientLayer也有startPoint和endPoint属性，他们决定了渐变的方向。这两个参数是以单位坐标系进行的定义，所以左上角坐标是{0, 0}，右下角坐标是{1, 1}。

小例子：简单的两种颜色的对角线渐变

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{
  		[super viewDidLoad];
  		// 创建梯度图层并添加这个图层到容器视图上
  		CAGradientLayer *gradientLayer = [CAGradientLayer layer];
  		gradientLayer.frame = self.containerView.bounds;
  		[self.containerView.layer addSublayer:gradientLayer];

  		// 设置梯度颜色
  		gradientLayer.colors = @[(__bridge id)[UIColor redColor].CGColor, (__bridge id)[UIColor blueColor].CGColor];

  		// 设置梯度开始和结束点
  		gradientLayer.startPoint = CGPointMake(0, 0);
  		gradientLayer.endPoint = CGPointMake(1, 1);
	}
	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.6.png)

####6.4.2、多重渐变
只要你想，colors属性可以包含很多颜色，创建一个彩虹一样的多重渐变也是很简单的。

默认情况下，这些颜色在空间上均匀地被渲染，但是我们可以用locations属性来调整空间。

locations属性是一个浮点数值的数组（以NSNumber类型封装）。这些浮点数定义了colors属性中每个不同颜色的位置，同样的，也是以单位坐标系进行标定。0.0代表着渐变的开始，1.0代表着结束。

locations数组并不是强制要求的，但是如果你给它赋值了就一定要确保locations的数组大小和colors数组大小一定要相同，否则你将会得到一个空白的渐变。

下面展示了一个基于之前对角线渐变的代码改造。现在变成了从红到黄最后到绿色的渐变。locations数组指定了0.0，0.25和0.5三个数值，这样这三个渐变就有点像挤在了左上角。

	- (void)viewDidLoad {
    	[super viewDidLoad];

    	// 创建梯度图层并添加到容器视图上
    	CAGradientLayer *gradientLayer = [CAGradientLayer layer];
    	gradientLayer.frame = self.containerView.bounds;
    	[self.containerView.layer addSublayer:gradientLayer];

    	// 设置梯度颜色
    	gradientLayer.colors = @[(__bridge id)[UIColor redColor].CGColor, (__bridge id [UIColor yellowColor].CGColor, (__bridge id)[UIColor greenColor].CGColor];

    	// 设置位置
    	gradientLayer.locations = @[@0.0, @0.25, @0.5];

    	// 设置梯度开始和结束点
    	gradientLayer.startPoint = CGPointMake(0, 0);
    	gradientLayer.endPoint = CGPointMake(1, 1);
	}
	
![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.7.png)

###6.5、CAReplicatorLayer

CAReplicatorLayer可以高效生成许多相似的图层。

它会绘制一个或多个图层的子图层，并在每个复制体上应用不同的变换。

####6.5.1、重复图层（Repeating Layers）

下面的例子，在屏幕的中间创建了一个小白色方块图层，然后用CAReplicatorLayer生成十个图层组成一个圆圈。

instanceCount属性指定了图层需要重复多少次。

instanceTransform指定了一个CATransform3D的变换（这种情况下，下一图层的位移和旋转将会移动到圆圈的下一个点）。

变换是逐步增加的，每个实例都是相对于前一实例布局。这就是为什么这些复制体最终不会出现在同意位置上。

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;

	@end

	@implementation ViewController
	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 创建一个重复图层并添加到视图上
    	CAReplicatorLayer *replicator = [CAReplicatorLayer layer];
    	replicator.frame = self.containerView.bounds;
    	[self.containerView.layer addSublayer:replicator];

    	// 设置重复次数
    	replicator.instanceCount = 10;

    	// 为每个实例应用变换
    	CATransform3D transform = CATransform3DIdentity;
    	transform = CATransform3DTranslate(transform, 0, 200, 0);
    	transform = CATransform3DRotate(transform, M_PI / 5.0, 0, 0, 1);
    	transform = CATransform3DTranslate(transform, 0, -200, 0);
    	replicator.instanceTransform = transform;

    	// 为每个实例设置颜色偏移
    	replicator.instanceBlueOffset = -0.1;
    	replicator.instanceGreenOffset = -0.1;

    	// 创建一个子图层并放置在重复图层中
    	CALayer *layer = [CALayer layer];
    	layer.frame = CGRectMake(100.0f, 100.0f, 100.0f, 100.0f);
    	layer.backgroundColor = [UIColor whiteColor].CGColor;
    	[replicator addSublayer:layer];
	}
	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.8.png)

	当图层在重复的时候，他们的颜色也在变化：这是用instanceBlueOffset和instanceGreenOffset属性实现的。
	
	通过逐步减少蓝色和绿色通道，我们逐渐将图层颜色转换成了红色。这个复制效果看起来很酷，但是CAReplicatorLayer真正应用到实际程序上的场景比如：一个游戏中导弹的轨迹云，或者粒子爆炸（尽管iOS 5已经引入了CAEmitterLayer，它更适合创建任意的粒子效果）。
	
除此之外，还有一个实际应用是：反射。

####6.5.2、镜像（反射）

使用CAReplicatorLayer并在一个复制图层上应用一个负比例变换，你就可以创建指定视图（或整个视图层次）内容的镜像图片，这样就创建了一个实时的镜像效果。

小例子：用CAReplicatorLayer自动绘制反射

	#import "ReflectionView.h"
	#import <QuartzCore/QuartzCore.h>

	@implementation ReflectionView

	+ (Class)layerClass
	{
    	return [CAReplicatorLayer class];
	}

	- (void)setUp
	{
    	// 配置重复图层
    	CAReplicatorLayer *layer = (CAReplicatorLayer *)self.layer;
    	layer.instanceCount = 2;

    	// 设置反射实例
    	CATransform3D transform = CATransform3DIdentity;
    	CGFloat verticalOffset = self.bounds.size.height + 2;
    	transform = CATransform3DTranslate(transform, 0, verticalOffset, 0);
    	transform = CATransform3DScale(transform, 1, -1, 0);
    	layer.instanceTransform = transform;

    	// 减少图层alpha
    	layer.instanceAlphaOffset = -0.6;
	}
￼
	- (id)initWithFrame:(CGRect)frame
	{
    	// 用代码创建时会调用
    	if ((self = [super initWithFrame:frame])) {
        	[self setUp];
    	}
    	return self;
	}

	- (void)awakeFromNib
	{
    	// 用IB创建时会调用
    	[self setUp];
	}
	@end
	
![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.9.png)

开源代码[ReflectionView](https://github.com/nicklockwood/ReflectionView)完成了一个自适应的渐变淡出效果（用CAGradientLayer和图层蒙板实现）

###6.6、CAScrollLayer

对于一个未转换的图层，它的bounds和它的frame是一样的，frame属性是由bounds属性自动计算而出的，所以更改任意一个值都会更新其他值。

但是如果你只想显示一个大图层里面的一小部分呢。比如说，你可能有一个很大的图片，你希望用户能够随意滑动，或者是一个数据或文本的长列表。在一个典型的iOS应用中，你可能会用到UITableView或是UIScrollView，但是对于独立的图层来说，什么会等价于刚刚提到的UITableView和UIScrollView呢？

之前介绍了图层的contentsRect属性的用法，它的确是能够解决在图层中小地方显示大图片的解决方法。但是如果你的图层包含子图层那它就不是一个非常好的解决方案，因为，这样做的话每次你想『滑动』可视区域的时候，你就需要手工重新计算并更新所有的子图层位置。

这个时候就需要CAScrollLayer了。

CAScrollLayer有一个-scrollToPoint:方法，它自动适应bounds的原点以便图层内容出现在滑动的地方。注意，这就是它做的所有事情。

前面提到过Core Animation并不处理用户输入，所以CAScrollLayer并不负责将触摸事件转换为滑动事件，既不渲染滚动条，也不实现任何iOS指定行为例如滑动反弹（当视图滑动超多了它的边界的将会反弹回正确的地方）。

下面的例子介绍了用CAScrollLayer来代替基础的UIScrollView。我们将会用CAScrollLayer作为视图的宿主图层，并创建一个自定义的UIView，然后用UIPanGestureRecognizer实现触摸事件响应。

	#import "ScrollView.h"
	#import <QuartzCore/QuartzCore.h> 
	@implementation ScrollView
	+ (Class)layerClass
	{
    	return [CAScrollLayer class];
	}

	- (void)setUp
	{
    	// 可切边
    	self.layer.masksToBounds = YES;

    	// 绑定手势
    	UIPanGestureRecognizer *recognizer = nil;
    	recognizer = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(pan:)];
    	[self addGestureRecognizer:recognizer];
	}

	- (id)initWithFrame:(CGRect)frame
	{
    	if ((self = [super initWithFrame:frame])) {
        	[self setUp];
    	}
    	return self;
	}

	- (void)awakeFromNib {
    	[self setUp];
	}

	- (void)pan:(UIPanGestureRecognizer *)recognizer
	{
    	// 计算偏移
    	CGPoint offset = self.bounds.origin;
    	offset.x -= [recognizer translationInView:self].x;
    	offset.y -= [recognizer translationInView:self].y;

    	// 滚动
    	[(CAScrollLayer *)self.layer scrollToPoint:offset];

    	// 重置
    	[recognizer setTranslation:CGPointZero inView:self];
	}
	@end
	
![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.10.png)

不同于UIScrollView，我们定制的滑动视图类并没有实现任何形式的边界检查（bounds checking）。图层内容极有可能滑出视图的边界并无限滑下去。

CAScrollLayer并没有等同于UIScrollView中contentSize的属性，所以当CAScrollLayer滑动的时候完全没有一个全局的可滑动区域的概念，也无法自适应它的边界原点至你指定的值。

它之所以不能自适应边界大小是因为它不需要，内容完全可以超过边界。

那你一定会奇怪用CAScrollLayer的意义到底何在，因为你可以简单地用一个普通的CALayer然后手动适应边界原点啊。真相其实并不复杂，UIScrollView并没有用CAScrollLayer，事实上，就是简单的通过直接操作图层边界来实现滑动。

CAScrollLayer有一个潜在的有用特性。如果你查看CAScrollLayer的头文件，你就会注意到有一个扩展分类实现了一些方法和属性：

	// 从图层树中查找并找到第一个可用的CAScrollLayer，然后滑动它使得指定点成为可视的。
	- (void)scrollPoint:(CGPoint)p; 
	// 实现了同样的事情只不过是作用在一个矩形上的
	- (void)scrollRectToVisible:(CGRect)r;
 	// 决定图层（如果存在的话）的哪部分是当前的可视区域。
 	@property(readonly) CGRect visibleRect;

如果你自己实现这些方法就会相对容易明白一点，但是CAScrollLayer帮你省了这些麻烦，所以当涉及到实现图层滑动的时候就可以用上了。

###6.7、CATiledLayer

有些时候你可能需要绘制一个很大的图片，常见的例子就是一个高像素的照片或者是地球表面的详细地图。

iOS应用通畅运行在内存受限的设备上，所以读取整个图片到内存中是不明智的。载入大图可能会相当地慢，那些对你看上去比较方便的做法（在主线程调用UIImage的-imageNamed:方法或者-imageWithContentsOfFile:方法）将会阻塞你的用户界面，至少会引起动画卡顿现象。

能高效绘制在iOS上的图片也有一个大小限制。所有显示在屏幕上的图片最终都会被转化为OpenGL纹理，同时OpenGL有一个最大的纹理尺寸（通常是2048 * 2048，或4096 * 4096，这个取决于设备型号）。

如果你想在单个纹理中显示一个比这大的图，即便图片已经存在于内存中了，你仍然会遇到很大的性能问题，因为Core Animation强制用CPU处理图片而不是更快的GPU。

`CATiledLayer`为载入大图造成的性能问题提供了一个解决方案：将大图分解成小片然后将他们单独按需载入。

####6.7.1、小片裁剪

这个示例中，我们将会从一个2048*2048分辨率的雪人图片入手。为了能够从CATiledLayer中获益，我们需要把这个图片裁切成许多小一些的图片。你可以通过代码来完成这件事情，但是如果你在运行时读入整个图片并裁切，那CATiledLayer这些所有的性能优点就损失殆尽了。理想情况下来说，最好能够逐个步骤来实现。

下面演示了一个简单的Mac OS命令行程序，它用CATiledLayer将一个图片裁剪成小图并存储到不同的文件中。

	#import <AppKit/AppKit.h>

	int main(int argc, const char * argv[])
	{
    	@autoreleasepool{
        	￼// 处理错误参数
        	if (argc < 2) {
            	NSLog(@"TileCutter arguments: inputfile");
            	return 0;
        	}

        	// 输入文件
        	NSString *inputFile = [NSString stringWithCString:argv[1] encoding:NSUTF8StringEncoding];

        	// 设置小片尺寸
        	CGFloat tileSize = 256; 
        	NSString *outputPath = [inputFile stringByDeletingPathExtension];

        	// 读取图片
        	NSImage *image = [[NSImage alloc] initWithContentsOfFile:inputFile];
        	NSSize size = [image size];
        	NSArray *representations = [image representations];
        	if ([representations count]){
            	NSBitmapImageRep *representation = representations[0];
            	size.width = [representation pixelsWide];
            	size.height = [representation pixelsHigh];
        	}
        	NSRect rect = NSMakeRect(0.0, 0.0, size.width, size.height);
        	CGImageRef imageRef = [image CGImageForProposedRect:&rect context:NULL hints:nil];

        	// 计算行数和列数
        	NSInteger rows = ceil(size.height / tileSize);
        	NSInteger cols = ceil(size.width / tileSize);

        	// 生成小片
        	for (int y = 0; y < rows; ++y) {
            	for (int x = 0; x < cols; ++x) {
            	// 解压小片图像
            	CGRect tileRect = CGRectMake(x*tileSize, y*tileSize, tileSize, tileSize);
            	CGImageRef tileImage = CGImageCreateWithImageInRect(imageRef, tileRect);

            	// 转换成jpeg数据
            	NSBitmapImageRep *imageRep = [[NSBitmapImageRep alloc] initWithCGImage:tileImage];
            	NSData *data = [imageRep representationUsingType: NSJPEGFileType properties:nil];
            	CGImageRelease(tileImage);

            	// 保存文件
            	NSString *path = [outputPath stringByAppendingFormat: @"_%02i_%02i.jpg", x, y];
            	[data writeToFile:path atomically:NO];
            	}
        	}
    	}
    	return 0;
	}

这个程序将2048 * 2048分辨率的雪人图案裁剪成了64个不同的256 * 256的小图。（256 * 256是CATiledLayer的默认小图大小，默认大小可以通过tileSize属性更改）。程序接受一个图片路径作为命令行的第一个参数。我们可以在编译的scheme将路径参数硬编码然后就可以在Xcode中运行了，但是以后作用在另一个图片上就不方便了。所以，我们编译了这个程序并把它保存到敏感的地方，然后从终端调用，如下面所示：

	> path/to/TileCutterApp path/to/Snowman.jpg

这个程序相当基础，但是能够轻易地扩展支持额外的参数比如小图大小，或者导出格式等等。运行结果是64个新图的序列，如下面命名：

	Snowman_00_00.jpg	
	Snowman_00_01.jpg
	Snowman_00_02.jpg
	...
	Snowman_07_07.jpg

既然我们有了裁切后的小图，我们就要让iOS程序用到他们。CATiledLayer很好地和UIScrollView集成在一起。除了设置图层和滑动视图边界以适配整个图片大小，我们真正要做的就是实现-drawLayer:inContext:方法，当需要载入新的小图时，CATiledLayer就会调用到这个方法。

小例子：一个简单的滚动CATiledLayer实现

	#import "ViewController.h"
	#import <QuartzCore/QuartzCore.h>

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIScrollView *scrollView;

	@end

	@implementation ViewController

	- (void)viewDidLoad
	{	
    	[super viewDidLoad];
    	// 添加小片图层
    	CATiledLayer *tileLayer = [CATiledLayer layer];￼
    	tileLayer.frame = CGRectMake(0, 0, 2048, 2048);
    	tileLayer.delegate = self; [self.scrollView.layer addSublayer:tileLayer];

    	// 配置滚动视图
    	self.scrollView.contentSize = tileLayer.frame.size;

    	// 绘制图层
    	[tileLayer setNeedsDisplay];
	}

	- (void)drawLayer:(CATiledLayer *)layer inContext:(CGContextRef)ctx
	{
    	//determine tile coordinate
    	CGRect bounds = CGContextGetClipBoundingBox(ctx);
    	NSInteger x = floor(bounds.origin.x / layer.tileSize.width);
    	NSInteger y = floor(bounds.origin.y / layer.tileSize.height);

    	//load tile image
    	NSString *imageName = [NSString stringWithFormat: @"Snowman_%02i_%02i, x, y];
    	NSString *imagePath = [[NSBundle mainBundle] pathForResource:imageName ofType:@"jpg"];
    	UIImage *tileImage = [UIImage imageWithContentsOfFile:imagePath];

    	// 绘制
    	UIGraphicsPushContext(ctx);
    	[tileImage drawInRect:bounds];
    	UIGraphicsPopContext();
	}
	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.11.png)

当你滑动这个图片，你会发现当CATiledLayer载入小图的时候，他们会淡入到界面中。这是CATiledLayer的默认行为。也可以用fadeDuration属性改变淡入时长或者直接禁用掉。

CATiledLayer（不同于大部分的UIKit和Core Animation方法）支持多线程绘制，-drawLayer:inContext:方法可以在多个线程中同时地并发调用，所以请确保这个方法中实现的绘制代码是线程安全的。

####6.7.2、Retina小图

这些小图并不是Retina的分辨率显示的。

为了以屏幕的原生分辨率来渲染CATiledLayer，我们需要设置图层的contentsScale来匹配UIScreen的scale属性：

	tileLayer.contentsScale = [UIScreen mainScreen].scale;

有趣的是，tileSize是以像素为单位，而不是点，所以增大了contentsScale就自动有了默认的小图尺寸（现在它是128 * 128的点而不是256 * 256）。

所以，我们不需要手工更新小图的尺寸或是在Retina分辨率下指定一个不同的小图。我们需要做的是适应小图渲染代码以对应安排scale的变化，然而：

	//determine tile coordinate
	CGRect bounds = CGContextGetClipBoundingBox(ctx);
	CGFloat scale = [UIScreen mainScreen].scale;
	NSInteger x = floor(bounds.origin.x / layer.tileSize.width * scale);
	NSInteger y = floor(bounds.origin.y / layer.tileSize.height * scale);

通过这个方法纠正scale也意味着我们的雪人图将以一半的大小渲染在Retina设备上（总尺寸是1024 * 1024，而不是2048 * 2048）。

这个通常都不会影响到用CATiledLayer正常显示的图片类型（比如照片和地图，他们在设计上就是要支持放大缩小，能够在不同的缩放条件下显示）。


###6.8、CAEmitterLayer

在iOS 5中，苹果引入了一个新的CALayer子类叫做CAEmitterLayer。

CAEmitterLayer是一个高性能的粒子引擎，被用来创建实时例子动画如：烟雾，火，雨等等这些效果。

一个小例子：用CAEmitterLayer创建爆炸效果

	#import "ViewController.h"
	#import <QuartzCore/QuartzCore.h>

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView;

	@end


	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    ￼
	    // 创建粒子发射图层
   		CAEmitterLayer *emitter = [CAEmitterLayer layer];
    	emitter.frame = self.containerView.bounds;
    	[self.containerView.layer addSublayer:emitter];

    	// 配置图层
    	emitter.renderMode = kCAEmitterLayerAdditive;
    	emitter.emitterPosition = CGPointMake(emitter.frame.size.width / 2.0, 		emitter.frame.size.height / 2.0);

    	// 创建一个粒子模板
    	CAEmitterCell *cell = [[CAEmitterCell alloc] init];
    	cell.contents = (__bridge id)[UIImage imageNamed:@"Spark.png"].CGImage;
    	cell.birthRate = 150;
    	cell.lifetime = 5.0;
    	cell.color = [UIColor colorWithRed:1 green:0.5 blue:0.1 alpha:1.0].CGColor;
    	cell.alphaSpeed = -0.4;
    	cell.velocity = 50;
    	cell.velocityRange = 50;
    	cell.emissionRange = M_PI * 2.0;

    	// 添加粒子模板到图层
    	emitter.emitterCells = @[cell];
	}
	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.12a.png)

`CAEMitterCell`的属性基本上可以分为三种：

- 粒子的某属性的初始值。比如，color属性指定了一个可以混合图片内容颜色的混合色。在示例中，我们将它设置为桔色。

- 粒子某属性的变化范围。比如emissionRange属性的值是2π，这意味着例子可以从360度任意位置反射出来。如果指定一个小一些的值，就可以创造出一个圆锥形

- 指定值在时间线上的变化。比如，在示例中，我们将alphaSpeed设置为-0.4，就是说例子的透明度每过一秒就是减少0.4，这样就有发射出去之后逐渐小时的效果。

CAEmitterLayer的属性它自己控制着整个例子系统的位置和形状。一些属性比如birthRate，lifetime和celocity，这些属性在CAEmitterCell中也有。这些属性会以相乘的方式作用在一起，这样你就可以用一个值来加速或者扩大整个例子系统。

其他值得提到的属性有以下这些：

	preservesDepth，是否将3D例子系统平面化到一个图层（默认值）或者可以在3D空间中混合其他的图层

	renderMode，控制着在视觉上粒子图片是如何混合的。例子中设置为kCAEmitterLayerAdditive。这样实现了这样一个效果：合并例子重叠部分的亮度使得看上去更亮。如果设置为默认的kCAEmitterLayerUnordered，效果就没那么好看了.

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.12.png)

###6.9、CAEAGLLayer

当iOS要处理高性能图形绘制时就是用OpenGL。

OpenGL提供了Core Animation的基础，它是底层的C接口，直接和iPhone，iPad的硬件通信，它没有对象或图层的概念，只是简单地处理三角形。

OpenGL中所有东西都是3D空间中有颜色和纹理的三角形。

为了能够以高性能使用Core Animation，需要判断到底需要绘制哪种内容（矢量图形，例子，文本等），然后选择合适的图层去呈现这些内容，不过Core Animation中只有部分类型的内容是被高度优化的；如果你想绘制的东西并不能找到标准的图层类，同时想要得到高性能就比较费事情了。因为OpenGL根本不会对你的内容进行假设，它能够绘制得相当快。

利用OpenGL，你可以绘制任何你知道必要的集合信息和形状逻辑的内容。所以很多游戏都喜欢用OpenGL（这些情况下，Core Animation的限制就明显了：它优化过的内容类型并不一定能满足需求），但是这样依赖，方便的高度抽象接口就没了。

在iOS 5中，苹果引入了一个新的框架叫做GLKit，它去掉了一些设置OpenGL的复杂性，提供了一个叫做CLKView的UIView的子类，帮你处理大部分的设置和绘制工作。前提是各种各样的OpenGL绘图缓冲的底层可配置项仍然需要你用CAEAGLLayer完成，它是CALayer的一个子类，用来显示任意的OpenGL图形。

大部分情况下都不需要手动设置CAEAGLLayer（假设用GLKView），现在的标准做法是设置一个OpenGL ES 2.0的上下文。

尽管不需要GLKit也可以做到这一切，但是GLKit囊括了很多额外的工作，比如设置顶点和片段着色器，同时在运行时载入到图形硬件中。编写GLSL代码和设置EAGLayer没有什么关系，所以可以用GLKBaseEffect类将着色逻辑抽象出来。

小例子：用CAEAGLLayer绘制一个三角形

	#import "ViewController.h"
	#import <QuartzCore/QuartzCore.h>
	#import <GLKit/GLKit.h>

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *glView;
	@property (nonatomic, strong) EAGLContext *glContext;
	@property (nonatomic, strong) CAEAGLLayer *glLayer;
	@property (nonatomic, assign) GLuint framebuffer;
	@property (nonatomic, assign) GLuint colorRenderbuffer;
	@property (nonatomic, assign) GLint framebufferWidth;
	@property (nonatomic, assign) GLint framebufferHeight;
	@property (nonatomic, strong) GLKBaseEffect *effect;
￼
	@end

	@implementation ViewController

	- (void)setUpBuffers
	{
    	//set up frame buffer
    	glGenFramebuffers(1, &_framebuffer);
    	glBindFramebuffer(GL_FRAMEBUFFER, _framebuffer);

    	//set up color render buffer
    	glGenRenderbuffers(1, &_colorRenderbuffer);
    	glBindRenderbuffer(GL_RENDERBUFFER, _colorRenderbuffer);
    	glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_RENDERBUFFER, _colorRenderbuffer);
    	[self.glContext renderbufferStorage:GL_RENDERBUFFER fromDrawable:self.glLayer];
    	glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_WIDTH, &_framebufferWidth);
    	glGetRenderbufferParameteriv(GL_RENDERBUFFER, GL_RENDERBUFFER_HEIGHT, &_framebufferHeight);

    	//check success
    	if (glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE) {
        	NSLog(@"Failed to make complete framebuffer object: %i", glCheckFramebufferStatus(GL_FRAMEBUFFER));
    	}
	}

	- (void)tearDownBuffers
	{
    	if (_framebuffer) {
        	//delete framebuffer
        	glDeleteFramebuffers(1, &_framebuffer);
        	_framebuffer = 0;
    	}

    	if (_colorRenderbuffer) {
        	//delete color render buffer
        	glDeleteRenderbuffers(1, &_colorRenderbuffer);
        	_colorRenderbuffer = 0;
    	}
	}

	- (void)drawFrame {
    	//bind framebuffer & set viewport
    	glBindFramebuffer(GL_FRAMEBUFFER, _framebuffer);
    	glViewport(0, 0, _framebufferWidth, _framebufferHeight);

    	//bind shader program
    	[self.effect prepareToDraw];

    	//clear the screen
    	glClear(GL_COLOR_BUFFER_BIT); glClearColor(0.0, 0.0, 0.0, 1.0);

    	//set up vertices
    	GLfloat vertices[] = {
        	-0.5f, -0.5f, -1.0f, 0.0f, 0.5f, -1.0f, 0.5f, -0.5f, -1.0f,
    	};

    	//set up colors
    	GLfloat colors[] = {
        	0.0f, 0.0f, 1.0f, 1.0f, 0.0f, 1.0f, 0.0f, 1.0f, 1.0f, 0.0f, 0.0f, 1.0f,
    	};

    	//draw triangle
    	glEnableVertexAttribArray(GLKVertexAttribPosition);
    	glEnableVertexAttribArray(GLKVertexAttribColor);
    	glVertexAttribPointer(GLKVertexAttribPosition, 3, GL_FLOAT, GL_FALSE, 0, vertices);
    	glVertexAttribPointer(GLKVertexAttribColor,4, GL_FLOAT, GL_FALSE, 0, colors);
    	glDrawArrays(GL_TRIANGLES, 0, 3);

    	//present render buffer
    	glBindRenderbuffer(GL_RENDERBUFFER, _colorRenderbuffer);
    	[self.glContext presentRenderbuffer:GL_RENDERBUFFER];
	}

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	//set up context
    	self.glContext = [[EAGLContext alloc] initWithAPI: kEAGLRenderingAPIOpenGLES2];
	    [EAGLContext setCurrentContext:self.glContext];

    	//set up layer
    	self.glLayer = [CAEAGLLayer layer];
    	self.glLayer.frame = self.glView.bounds;
    	[self.glView.layer addSublayer:self.glLayer];
    	self.glLayer.drawableProperties = @{kEAGLDrawablePropertyRetainedBacking:@NO, kEAGLDrawablePropertyColorFormat: kEAGLColorFormatRGBA8};

    	//set up base effect
    	self.effect = [[GLKBaseEffect alloc] init];

    	//set up buffers
    	[self setUpBuffers];

    	//draw frame
    	[self drawFrame];
	}

	- (void)viewDidUnload
	{
    	[self tearDownBuffers];
    	[super viewDidUnload];
	}

	- (void)dealloc
	{
    	[self tearDownBuffers];
    	[EAGLContext setCurrentContext:nil];
	}
	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.13.png)

在一个真正的OpenGL应用中，我们可能会用NSTimer或CADisplayLink周期性地每秒钟调用-drawRrame方法60次，同时会将几何图形生成和绘制分开以便不会每次都重新生成三角形的顶点（这样也可以让我们绘制其他的一些东西而不是一个三角形而已），不过上面这个例子已经足够演示了绘图原则了。

###6.10、AVPlayerLayer

AVPlayerLayer不是Core Animation框架的一部分，它是由AVFoundation提供的，它和Core Animation紧密地结合在一起，提供了一个CALayer子类来显示自定义的内容类型。

AVPlayerLayer是用来在iOS上播放视频的。

它是高级接口例如MPMoivePlayer的底层实现，提供了显示视频的底层控制。

AVPlayerLayer的使用相当简单：

	用+playerLayerWithPlayer:方法创建一个已经绑定了视频播放器的图层
	
	或者可以先创建一个图层，然后用player属性绑定一个AVPlayer实例。

小例子：用AVPlayerLayer播放视频

	#import "ViewController.h"
	#import <QuartzCore/QuartzCore.h>
	#import <AVFoundation/AVFoundation.h>

	@interface ViewController ()

	@property (nonatomic, weak) IBOutlet UIView *containerView; @end

	@implementation ViewController

	- (void)viewDidLoad
	{
    	[super viewDidLoad];
    	// 获取视频URL
    	NSURL *URL = [[NSBundle mainBundle] URLForResource:@"Ship" withExtension:@"mp4"];

    	// 创建播放器和播放器图层
    	AVPlayer *player = [AVPlayer playerWithURL:URL];
    	AVPlayerLayer *playerLayer = [AVPlayerLayer playerLayerWithPlayer:player];

    	// 设置播放器图层的frame并绑定在视图上
    	playerLayer.frame = self.containerView.bounds;
    	[self.containerView.layer addSublayer:playerLayer];

    	// 播放视频
    	[player play];
	}
	@end

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.14.png)

用代码创建了一个AVPlayerLayer，把它添加到了一个容器视图中，而不是直接在controller中的主视图上添加。

这样是为了使得图层在最中间；因为Core Animation并不支持自动大小和自动布局，一旦设备被旋转了我们就要手动重新放置位置。

又因为AVPlayerLayer是CALayer的子类，它继承了父类的所有特性。

小例子 给视频增加变换，边框和圆角

	- (void)viewDidLoad
	{
    	...
    	// 设置播放器图层frame并绑定在视图上
    	playerLayer.frame = self.containerView.bounds;
    	[self.containerView.layer addSublayer:playerLayer];

 	   	// 变换图层
   	 	CATransform3D transform = CATransform3DIdentity;
    	transform.m34 = -1.0 / 500.0;
    	transform = CATransform3DRotate(transform, M_PI_4, 1, 1, 0);
    	playerLayer.transform = transform;
    ￼
    	// 添加圆角和边框
    	playerLayer.masksToBounds = YES;
    	playerLayer.cornerRadius = 20.0;
    	playerLayer.borderColor = [UIColor redColor].CGColor;
    	playerLayer.borderWidth = 5.0;

    	// 播放视频
    	[player play];
	}
![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvanced6.15.png)



