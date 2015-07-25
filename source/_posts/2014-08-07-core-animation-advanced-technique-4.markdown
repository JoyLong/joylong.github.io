---
layout: post
title: "Core Animation Advanced Technique 学习笔记(4)"
date: 2014-08-07 00:35:15 +0800
comments: true
categories: iOS-CoreAnimation
---


#第一部分：下面的图层
##5.变换（Transforms）

###5.1、仿射变换

####5.1.1、仿射变换介绍

之前的例子里有一个钟表的指针，使用了UIView的`transform`属性旋转了指针

UIView的transform属性是CGAffineTransform类型，用于在二维空间做`旋转`，`缩放`和`平移`。

CGAffineTransform是一个可以和二维空间向量（例如CGPoint）做乘法变换运算的3X2的矩阵

将CGPoint的每一列和CGAffineTransform矩阵的每一行对应元素相乘再求和，就形成了一个新的CGPoint类型的结果。

**但是，CGPoint是1X2的 CGAffineTRansform是3X2的矩阵，没法做矩阵乘法，于是必须要给矩阵填充一些值，这些值主要是让矩阵做乘法，但是不影响运算结果，也不会发生变化**

如下图：

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.1.png)


当对图层应用变换矩阵，图层矩形内的每一个点都被相应地做变换，从而形成一个新的四边形的形状。

CGAffineTransform中的“仿射”的意思是无论变换矩阵用什么值，图层中平行的两条线在变换之后任然保持平行，CGAffineTransform可以做出任意符合规则的变换，

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.2.png)

####5.1.2、创建CGAffineTransform

Core Graphics提供了一系列函数，让毫无数学基础的开发者能够简单地做一些变换。

创建了一个CGAffineTransform实例可以用如下几个函数：

```
CGAffineTransformMakeRotation(CGFloat angle)  // 旋转一个角度
CGAffineTransformMakeScale(CGFloat sx, CGFloat sy) // 缩放
CGAffineTransformMakeTranslation(CGFloat tx, CGFloat ty) // 平移，每个点都平移一定距离
```

**小例子：把一个图旋转45度**

UIView可以通过设置transform属性做变换，但实际上它只是封装了内部图层的变换。

CALayer同样也有一个transform属性，但它的类型是CATransform3D，而不是CGAffineTransform。

CALayer对应于UIView的transform属性叫做`affineTransform`，

下面的例子是使用affineTransform对图层旋转45度

```
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *layerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    // 旋转图层45度(这里用的是弧度)
    CGAffineTransform transform = CGAffineTransformMakeRotation(M_PI_4);
    self.layerView.layer.affineTransform = transform;
}

@end
```

注：上面旋转设置的都是旋转弧度，可以用以下的宏做换算：
	
	#define RADIANS_TO_DEGREES(x) ((x)/M_PI*180.0) 
	#define DEGREES_TO_RADIANS(x) ((x)/180.0*M_PI)

####5.1.3、组合变换（Combining Transforms）

旋转，缩放和平移，我们可能不止对图片做一个操作，可能会同时都变换，因此需要创建一个CGAffineTransform类型的空值，矩阵论中称作单位矩阵

Core Graphics同样也提供了一个方便的常量：

	CGAffineTransformIdentity
		
如果需要混合两个已经存在的变换矩阵，就可以使用如下方法，在两个变换的基础上创建一个新的变换：

	CGAffineTransformConcat(CGAffineTransform t1, CGAffineTransform t2);
	
**小例子，先缩小50%，再旋转30度，最后向右移动200个像素**

```
- (void)viewDidLoad
{
    [super viewDidLoad];
    // 创建一个新的变换，单位矩阵
    CGAffineTransform transform = CGAffineTransformIdentity; 
    // 缩小50%
    transform = CGAffineTransformScale(transform, 0.5, 0.5); 
    // 旋转30度
    transform = CGAffineTransformRotate(transform, M_PI / 180.0 * 30.0); 
    // 平移200点
    transform = CGAffineTransformTranslate(transform, 200, 0);
    // 应用到图层上
    self.layerView.layer.affineTransform = transform;
}
```

####5.1.4、剪裁变换(Shear Transform)

Core Graphics为你提供了计算变换矩阵的一些方法，所以很少需要直接设置CGAffineTransform的值。

除非需要创建一个斜切的变换，但是Core Graphics并没有提供直接的函数。

斜切变换是仿射变换的第四种类型，较于平移，旋转和缩放并不常用（这也是Core Graphics没有提供相应函数的原因），但有些时候也会很有用。

这个变换的效果如下图

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.3.png)

```
@implementation ViewController

CGAffineTransform CGAffineTransformMakeShear(CGFloat x, CGFloat y)
{
    CGAffineTransform transform = CGAffineTransformIdentity;
    transform.c = -x;
    transform.b = y;
    return transform;
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    // 倾斜图层到45度
    self.layerView.layer.affineTransform = CGAffineTransformMakeShear(1, 0);
}

@end
```

##5.2、3D变换

###5.2.1、3D变换介绍

Core Graphics实际上是一个2D绘图API，并且CGAffineTransform仅仅对2D变换有效。

之前有提到了zPosition属性，可以用来让图层看起来靠近或远离用户，但是transform属性（CATransform3D类型）可以真正让图层在3D空间内移动或者旋转。

和CGAffineTransform类似，CATransform3D也是一个矩阵，但是和2x3的矩阵不同，CATransform3D是一个可以在3维空间内做变换的4x4的矩阵。

下图是对一个3D点做`CATransform3D`的矩阵变换

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.4.png)

Core Animation提供了一系列的方法用来创建和组合CATransform3D类型的矩阵，和Core Graphics的函数类似，但是3D的平移和旋转多处了一个z参数，并且旋转函数除了angle之外多出了x,y,z三个参数，分别决定了每个坐标轴方向上的旋转：

	CATransform3DMakeRotation(CGFloat angle, CGFloat x, CGFloat y, CGFloat z)
	CATransform3DMakeScale(CGFloat sx, CGFloat sy, CGFloat sz) 
	CATransform3DMakeTranslation(Gloat tx, CGFloat ty, CGFloat tz)
	
下图为XYZ轴位置以及相应的旋转

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.5.png)

由图所见，绕Z轴的旋转等同于之前二维空间的仿射旋转，但是绕X轴和Y轴的旋转就突破了屏幕的二维空间，并且在用户视角看来发生了倾斜。


**绕Y轴旋转图层**

```
@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //rotate the layer 45 degrees along the Y axis
    CATransform3D transform = CATransform3DMakeRotation(M_PI_4, 0, 1, 0);
    self.layerView.layer.transform = transform;
}

@end
```

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.6.png)

看起来图层并没有被旋转，而是仅仅在水平方向上的一个压缩，这是因为我们在用一个斜向的视角看，而不是透视视角。

###5.2.2、透视投影
####5.2.2.1、透视投影介绍

在真实世界中，东西离我们越远看起来就越小

在等距投影中，远处的物体和近处的物体保持同样的缩放比例

为了做一些修正，我们需要引入投影变换（又称作z变换）来对除了旋转之外的变换矩阵做一些修改，Core Animation并没有给我们提供设置透视变换的函数，因此我们需要手动修改矩阵值。

CATransform3D的透视效果通过矩阵中m34的值来控制。

**m34用于按比例缩放X和Y的值来计算到底要离视角多远。**


![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.7.png)

m34的默认值是0，我们可以通过设置m34为(-1.0 / d)来应用透视效果，d代表了视角相机和屏幕之间的距离，以像素为单位，那应该如何计算这个距离呢？大概估算一个就行了。

因为视角相机实际上并不存在，所以可以根据屏幕上的显示效果自由决定它放置的位置。通常500-1000就已经很好了，但对于特定的图层有时候更小后者更大的值会看起来更舒服，减少距离的值会增强透视效果，所以一个非常微小的值会让它看起来更加失真，然而一个非常大的值会让它基本失去透视效果。

```
@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    // 创建一个新的transform
    CATransform3D transform = CATransform3DIdentity;
    // 应用透视
    transform.m34 = - 1.0 / 500.0;
    // 绕Y轴旋转45度
    transform = CATransform3DRotate(transform, M_PI_4, 0, 1, 0);
    // 应用到图层上
    self.layerView.layer.transform = transform;
}

@end
```

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.8.png)

####5.2.2.2、消亡点

当在透视角度绘图的时候，远离相机视角的物体将会变小变远，当远离到一个极限距离，它们可能就缩成了一个点，于是所有的物体最后都汇聚消失在同一个点。

在现实中，这个点通常是视图的中心，于是为了在应用中创建拟真效果的透视，这个店应该聚在屏幕中点，或者至少是包含所有3D对象的视图中点。

Core Animation定义了这个点位于变换图层的anchorPoint。这就是说，当图层发生变换时，这个点永远位于图层变换之前anchorPoint的位置。

当改变一个图层的position，你也改变了它的消亡点，做3D变换的时候要时刻记住，当你视图通过调整m34来让它更加有3D效果，应该首先把它放置于屏幕中央，然后通过平移来把它移动到指定位置（而不是直接改变它的position），这样所有的3D图层都共享一个消亡点。

####5.2.2.3、sublayerTransform

如果有多个视图或者图层，每个都要做3D变换，那就需要分别设置相同的m34值，并且确保在变换之前都在屏幕中央共享同一个position。

CALayer有一个属性叫做`sublayerTransform`。它也是CATransform3D类型，但和对一个图层的变换不同，它影响到所有的子图层。

这意味着你可以一次性对包含这些图层的容器做变换，于是所有的子图层都自动继承了这个变换方法。

通过在一个地方设置透视变换会很方便，同时它会带来另一个显著的优势：

	消亡点被设置在容器图层的中点，从而不需要再对子图层分别设置了。
	
	这意味着你可以随意使用position和frame来放置子图层，而不需要把它们放置在屏幕中点，然后为了保证统一的消亡点用变换来做平移。


**小例子：应用sublayerTransform**

```
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;
@property (nonatomic, weak) IBOutlet UIView *layerView1;
@property (nonatomic, weak) IBOutlet UIView *layerView2;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    //apply perspective transform to container
    CATransform3D perspective = CATransform3DIdentity;
    perspective.m34 = - 1.0 / 500.0;
    self.containerView.layer.sublayerTransform = perspective;
    //rotate layerView1 by 45 degrees along the Y axis
    CATransform3D transform1 = CATransform3DMakeRotation(M_PI_4, 0, 1, 0);
    self.layerView1.layer.transform = transform1;
    //rotate layerView2 by 45 degrees along the Y axis
    CATransform3D transform2 = CATransform3DMakeRotation(-M_PI_4, 0, 1, 0);
    self.layerView2.layer.transform = transform2;
}
```

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.9.png)

####5.2.2.4、背面

既然可以在3D场景下旋转图层，那么旋转180度将会把图层完全旋转一个半圈，于是完全背对了相机视角。

视图的背面，就是一个镜像对称的图片

CALayer有一个叫做`doubleSided`的属性来控制图层的背面是否要被绘制。这是一个BOOL类型，默认为YES，如果设置为NO，那么当图层正面从相机视角消失的时候，它将不会被绘制。


####5.2.2.5、扁平化图层

如果对包含已经做过变换的图层的图层做反方向的变换将会发什么什么呢？是不是有点困惑？

如果内部图层相对外部图层做了相反的变换（这里是绕Z轴的旋转），那么按照逻辑这两个变换将被相互抵消。

```
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *outerView;
@property (nonatomic, weak) IBOutlet UIView *innerView;

@end

@implementation ViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    // 外层旋转45度
    CATransform3D outer = CATransform3DMakeRotation(M_PI_4, 0, 0, 1);
    self.outerView.layer.transform = outer;
    // 内层旋转-45度
    CATransform3D inner = CATransform3DMakeRotation(-M_PI_4, 0, 0, 1);
    self.innerView.layer.transform = inner;
}

@end
```

代码执行后的样子：

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.10.png)


修改代码，让内外两个视图绕Y轴旋转而不是Z轴，再加上透视效果，以便我们观察。注意不能用sublayerTransform属性，因为内部的图层并不直接是容器图层的子图层，所以这里分别对图层设置透视变换

```
- (void)viewDidLoad
{
    [super viewDidLoad];
    //rotate the outer layer 45 degrees
    CATransform3D outer = CATransform3DIdentity;
    outer.m34 = -1.0 / 500.0;
    outer = CATransform3DRotate(outer, M_PI_4, 0, 1, 0);
    self.outerView.layer.transform = outer;
    //rotate the inner layer -45 degrees
    CATransform3D inner = CATransform3DIdentity;
    inner.m34 = -1.0 / 500.0;
    inner = CATransform3DRotate(inner, -M_PI_4, 0, 1, 0);
    self.innerView.layer.transform = inner;
}
```

预期的效果如下

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.11.png)


相反，我们看到的结果如下图所示。

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.12.png)

这是由于Core Animation图层存在于3D空间之内，但它们并不都存在同一个3D空间。每个图层的3D场景其实是扁平化的，当你从正面观察一个图层，看到的实际上由子图层创建的想象出来的3D场景，但当你倾斜这个图层，你会发现实际上这个3D场景仅仅是被绘制在图层的表面。

当你在玩一个3D游戏，实际上仅仅是把屏幕做了一次倾斜，或许在游戏中可以看见有一面墙在你面前，但是倾斜屏幕并不能够看见墙里面的东西。所有场景里面绘制的东西并不会随着你观察它的角度改变而发生变化；图层也是同样的道理。

这使得用Core Animation创建非常复杂的3D场景变得十分困难。

你不能够使用图层树去创建一个3D结构的层级关系--在相同场景下的任何3D表面必须和同样的图层保持一致，这是因为每个的父视图都把它的子视图扁平化了。

CALayer有一个叫做`CATransformLayer`的子类来解决这个问题。

##5.3、固体对象 

###5.3.1、创建一个立方体
现在来试着创建一个固态的3D对象（实际上是一个技术上所谓的空洞对象，但它以固态呈现）。我们用六个独立的视图来构建一个立方体的各个面。


**6个View创建一个立方体**

```
@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;
@property (nonatomic, strong) IBOutletCollection(UIView) NSArray *faces;

@end

@implementation ViewController

- (void)addFace:(NSInteger)index withTransform:(CATransform3D)transform
{
    // 获取立方体的面并添加到容器上
    UIView *face = self.faces[index];
    [self.containerView addSubview:face];
    // 在容器里居中这些面
    CGSize containerSize = self.containerView.bounds.size;
    face.center = CGPointMake(containerSize.width / 2.0, containerSize.height / 2.0);
    // 应用这些变换
    face.layer.transform = transform;
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    // 设置容器子图层的变换
    CATransform3D perspective = CATransform3DIdentity;
    perspective.m34 = -1.0 / 500.0;
    self.containerView.layer.sublayerTransform = perspective;
    // 增加立方体面1
    CATransform3D transform = CATransform3DMakeTranslation(0, 0, 100);
    [self addFace:0 withTransform:transform];
    // 增加立方体面2
    transform = CATransform3DMakeTranslation(100, 0, 0);
    transform = CATransform3DRotate(transform, M_PI_2, 0, 1, 0);
    [self addFace:1 withTransform:transform];
    // 增加立方体面3
    transform = CATransform3DMakeTranslation(0, -100, 0);
    transform = CATransform3DRotate(transform, M_PI_2, 1, 0, 0);
    [self addFace:2 withTransform:transform];
    // 增加立方体面4
    transform = CATransform3DMakeTranslation(0, 100, 0);
    transform = CATransform3DRotate(transform, -M_PI_2, 1, 0, 0);
    [self addFace:3 withTransform:transform];
    // 增加立方体面5
    transform = CATransform3DMakeTranslation(-100, 0, 0);
    transform = CATransform3DRotate(transform, -M_PI_2, 0, 1, 0);
    [self addFace:4 withTransform:transform];
    // 增加立方体面6
    transform = CATransform3DMakeTranslation(0, 0, -100);
    transform = CATransform3DRotate(transform, M_PI, 0, 1, 0);
    [self addFace:5 withTransform:transform];
}

@end
```
![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.13.png)

只有一面根本看不出来是立方体

所以我们旋转一下

但是，这个立方体是6个面组成的，正常逻辑就是旋转立方体，这样就必须要旋转6个面

我们还有另外一个简单的方法：调整容器视图的`sublayerTransform`去旋转照相机

为`containerView`的layer的`perspective`增加两行代码

```
// 绕X轴旋转45度
perspective = CATransform3DRotate(perspective, -M_PI_4, 1, 0, 0);  
// 绕Y轴旋转45度
perspective = CATransform3DRotate(perspective, -M_PI_4, 0, 1, 0);
```

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.14.png)

###5.3.2、为立方体加上光亮和阴影

现在它看起来更像是一个立方体了，但是每个面之间的连接处很难分辨，这是因为没有阴影和光照，显得很不真实。

如果想让立方体看起来更加真实，需要自己做一个阴影效果。

可以通过改变每个面的背景颜色或者直接用带光亮效果的图片来调整。

如果要动态创建光线效果，可以根据每个视图的方向应用不同的alpha值做出半透明的阴影图层，但为了计算阴影图层的不透明度，就需要得到每个面垂直于表面的向量，然后根据一个你自己假设的光源来计算出两个向量叉乘结果。

叉乘代表了光源和图层之间的角度，从而决定了它有多大程度上的光亮。

**下面的例子用GLKit框架来做向量的计算（你需要引入GLKit库来运行代码），每个面的CATransform3D都被转换成GLKMatrix4，然后通过GLKMatrix4GetMatrix3函数得出一个3×3的旋转矩阵。这个旋转矩阵指定了图层的方向，然后可以用它来得到这个垂直与表面的向量的值。**

试着调整`LIGHT_DIRECTION`和`AMBIENT_LIGHT`的值来切换光线效果

```
#import "ViewController.h" 
#import <QuartzCore/QuartzCore.h> 
#import <GLKit/GLKit.h>

#define LIGHT_DIRECTION 0, 1, -0.5 
#define AMBIENT_LIGHT 0.5

@interface ViewController ()

@property (nonatomic, weak) IBOutlet UIView *containerView;
@property (nonatomic, strong) IBOutletCollection(UIView) NSArray *faces;

@end

@implementation ViewController

- (void)applyLightingToFace:(CALayer *)face
{
    //add lighting layer
    CALayer *layer = [CALayer layer];
    layer.frame = face.bounds;
    [face addSublayer:layer];
    //convert the face transform to matrix
    //(GLKMatrix4 has the same structure as CATransform3D)
    CATransform3D transform = face.transform;
    GLKMatrix4 matrix4 = *(GLKMatrix4 *)&transform;
    GLKMatrix3 matrix3 = GLKMatrix4GetMatrix3(matrix4);
    //get face normal
    GLKVector3 normal = GLKVector3Make(0, 0, 1);
    normal = GLKMatrix3MultiplyVector3(matrix3, normal);
    normal = GLKVector3Normalize(normal);
    //get dot product with light direction
    GLKVector3 light = GLKVector3Normalize(GLKVector3Make(LIGHT_DIRECTION));
    float dotProduct = GLKVector3DotProduct(light, normal);
    //set lighting layer opacity
    CGFloat shadow = 1 + dotProduct - AMBIENT_LIGHT;
    UIColor *color = [UIColor colorWithWhite:0 alpha:shadow];
    layer.backgroundColor = color.CGColor;
}

- (void)addFace:(NSInteger)index withTransform:(CATransform3D)transform
{
    //get the face view and add it to the container
    UIView *face = self.faces[index];
    [self.containerView addSubview:face];
    //center the face view within the container
    CGSize containerSize = self.containerView.bounds.size;
    face.center = CGPointMake(containerSize.width / 2.0, containerSize.height / 2.0);
    // apply the transform
    face.layer.transform = transform;
    //apply lighting
    [self applyLightingToFace:face.layer];
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    //set up the container sublayer transform
    CATransform3D perspective = CATransform3DIdentity;
    perspective.m34 = -1.0 / 500.0;
    perspective = CATransform3DRotate(perspective, -M_PI_4, 1, 0, 0);
    perspective = CATransform3DRotate(perspective, -M_PI_4, 0, 1, 0);
    self.containerView.layer.sublayerTransform = perspective;
    //add cube face 1
    CATransform3D transform = CATransform3DMakeTranslation(0, 0, 100);
    [self addFace:0 withTransform:transform];
    //add cube face 2
    transform = CATransform3DMakeTranslation(100, 0, 0);
    transform = CATransform3DRotate(transform, M_PI_2, 0, 1, 0);
    [self addFace:1 withTransform:transform];
    //add cube face 3
    transform = CATransform3DMakeTranslation(0, -100, 0);
    transform = CATransform3DRotate(transform, M_PI_2, 1, 0, 0);
    [self addFace:2 withTransform:transform];
    //add cube face 4
    transform = CATransform3DMakeTranslation(0, 100, 0);
    transform = CATransform3DRotate(transform, -M_PI_2, 1, 0, 0);
    [self addFace:3 withTransform:transform];
    //add cube face 5
    transform = CATransform3DMakeTranslation(-100, 0, 0);
    transform = CATransform3DRotate(transform, -M_PI_2, 0, 1, 0);
    [self addFace:4 withTransform:transform];
    //add cube face 6
    transform = CATransform3DMakeTranslation(0, 0, -100);
    transform = CATransform3DRotate(transform, M_PI, 0, 1, 0);
    [self addFace:5 withTransform:transform];
}

@end
```

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.15.png)

###5.3.3、处理立方体的点击事件

目前这个立方体点击了没反应，这不是因为响应事件没被处理，而是在于视图的顺序

之前提过，点击事件的处理由视图在父视图中的顺序决定的，而不是3D空间中的Z轴顺序。

当给立方体添加视图的时候，按照视图/图层顺序来说，4，5，6在3的前面。

即使我们看不见4，5，6的表面（因为被1，2，3遮住了），iOS在事件响应上仍然保持之前的顺序。

当试图点击表面3上的按钮，表面4，5，6截断了点击事件（取决于点击的位置），这就和普通的2D布局在按钮上覆盖物体一样。

你也许认为把`doubleSided`设置成NO可以解决这个问题，因为它不再渲染视图后面的内容，但实际上并不起作用。

因为背对相机而隐藏的视图仍然会响应点击事件（这和通过设置hidden属性或者设置alpha为0而隐藏的视图不同，那两种方式将不会响应事件）。

所以即使禁止了双面渲染仍然不能解决这个问题（虽然由于性能问题，还是需要把它设置成NO）。

但是有几种正确的方案：

**1、把除了表面3的其他视图userInteractionEnabled属性都设置成NO来禁止事件传递。**

**2、简单通过代码把视图3覆盖在视图6上。**

![Mou icon](http://github-octopress.qiniudn.com/CoreAnimationAdvancedTechnique5.16.png)
