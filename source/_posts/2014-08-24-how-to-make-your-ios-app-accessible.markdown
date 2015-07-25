---
layout: post
title: "如何用Accessibility让你的应用可以为盲人服务"
date: 2014-08-24 11:56:01 +0800
comments: true
categories: iOS
---

##前言

现在市面上iOS应用非常多，随着苹果设备的普及，很多残障人士也开始使用苹果设备，但是随之而来的问题就是很多应用并不能让他们方便自如的使用。

其实像苹果，谷歌这样的厂商，已经为残障人士做了很多很多，只是我们平时不关注这些方面，于是感觉有些无能为力。但是其实让app也能为残障人士使用是一件非常非常简单的事情。大方向上苹果已经为开发者铺好了路，剩下的只是开发者如何去做而已。

##VoiceOver介绍

VoiceOver是苹果推出的语音辅助功能，让用户不需要看屏幕就能控制设备。在iOS 3.0时代，苹果就提供了VoiceOver来帮助用户听程序。这个功能可以帮助盲人无障碍的使用手机。VoiceOver可以描述一个应用的界面并通过声音来帮助用户使用程序。当VoiceOver打开时，用户不需要担心突然删除一个联系人或者打电话出去，因为VoiceOver会告诉用户现在在哪个位置，在进行什么操作，有什么样的结果。

苹果对VoiceOver的态度是：iOS3.0以后的app本应该支持VoiceOver，方便用户使用。

为此iOS和iOS SDK做了一些事情：

- 让标准UIKit控件和视图默认能够用VoiceOver访问

- 提供可访问的UI界面，而且定义了标准化流程

- 提供工具来提高易访问性以及测试


#####当然，如果要问为什么要支持VoiceOver？

	好处自然有很多，扩大用户人群啦，通过语音即可控制啦，政治正确的事情啦，等等等等，不一而足

	而你付出的无非只是很少很少的一点点时间，何乐不为呢？而且残疾人已经够不幸了，为这个社会做一点贡献，其实也挺好的嘛。


**苹果提供了Accessibility相关方法来提供相应支持**

**一个app是易访问的，主要表现在所有的用户接口元素都能够与用户交互。一个用户接口元素是易访问的，主要表现在它能够作为一个易访问元素报告自己。**

**一个易访问的接口元素必须提供关于屏幕位置，名称，行为，值和类型的准确和有用的信息。这些信息将通过VoiceOver告诉用户。**


##Accessibility API介绍

iOS 3.0之后就提供了Accessibility的接口，这个是帮助程序提供信息给VoiceOver，以便盲人使用

UI易访问编程接口是UIKit的一部分，并且默认已经被UIKit的控件和视图实现。这意味着当你使用标准控件和视图，苹果已经帮你做了很多很多事情了，要让程序变得易访问变得不是那么的难了。

iOS SDK也提供了工具帮助你使你的应用程序易访问：

- IB提供简便的方式来设置可被描述的信息。
- 易访问性检查，是显示嵌入在app的用户界面中的易访问性信息，并且允许在iOS模拟器上运行的时候验证这些信息。

此外，可以使用VoiceOver自己来测试应用的易访问性。如何测试的部分请参见苹果文档[Verifying App Accessibility on iOS](https://developer.apple.com/library/ios/technotes/TestingAccessibilityOfiOSApps/TestAccessibilityonYourDevicewithVoiceOver/TestAccessibilityonYourDevicewithVoiceOver.html#//apple_ref/doc/uid/TP40012619-CH3-SW4)，基本都是演示的图，跟着做就可以了


###UI易访问性编程接口

UI易访问性变成接口又两个协议，一个类，一个函数和一系列常量组成：

- UIAccessibility协议. 对象实现UIAccessibility协议。实现UIAccessibility协议的对象会报告它们的访问性状态(也就是说，是否可访问)以及提供关于自己的描述信息。默认情况，标准UIKit控件和视图都会实现UIAccessibility协议。

- UIAccessibilityContainer协议. 这个协议允许UIView的子类让它自身的部分或者全部元素像单独的元素一样易访问。当元素被包含在一个Viwe而不是自己是UIView的子类的时候，元素不会自动带有易访问性，这个协议特别有用.

- UIAccessibilityElement类. 这个类定义一个通过UIAccessibilityContainer协议返回的对象。你可以创建一个UIAccessibilityElement实例来表示一个没有自动获得易访问性的item，例如一个对象没有继承自UIView，或者一个不存在的对象。

- UIAccessibilityConstants.h头文件. 这个头文件定义了很多常量，这些常量描述易访问性元素能够展现的特性。也定义了很多应用程序能够发送的通知。

###易访问性属性

描述一个易访问用户接口元素的属性组成了UI Accessibility API的核心。当用户和一个控件或视图做交互时，VoiceOver为用户提供属性信息.

属性也是编程接口的一部分，属性封装的信息可以区分不同的控件和视图。对于标准UIKit控件和视图，可能需要确保默认属性信息是适合你的app。对于自定义控件和视图，可能要提供大部分属性信息。

UI易访问性编程接口定义了下列属性：

- Label. 一个简短的，本地化文本或者段落来描述控件或视图，但是不能标识这个元素的类型。例如"Add"或"Play"

- Traits. 一个或多个独立的特征、每个特征描述一个元素状态、行为、用途的单独表现。

- Hint. 一个简短的本地化的段落来描述元素行为造成的结果，例如添加一个标题，或者打开购物清单

- Frame. 屏幕上元素的frame,这个是通过CGRect的结构体来指明一个元素的位置和大小

- Value. 一个元素的当前值，当这个值不能通过label表示时，例如，一个侧边栏的label表示速度，但是值可能是50%

易访问性元素提供属性的内容，这个内容可能是默认的也可能是开发者提供的。一个易访问性元素总是提供frame和label属性的内容。frame属性是必须的，因为一个易访问性数据必须总是能够报告其位置（注：如果是继承自UIView则默认是有frame属性的）。label属性也是必备的，因为包含VoiceOver朗读时的元素的名字或描述

一个易访问性元素不一定要提供hint和traits属性，例如一个元素没有实施行为则不需要提供hint.

一个易访问性元素只有当元素的内容是可改变的并且不能被label很好描述时，才要给value属性提供信息。例如，一个text field包含一个邮箱地址，那么label可能是“邮箱地址” 但是这个的内容是依赖用户输入，格式也会是“username@address”


**下图是VoiceOver读出来的文本**

![Mou icon](http://github-octopress.qiniudn.com/accessibility_1.jpg)

VoiceOver的用户依赖他们听到的label和hint来使用app。因此，让每个控件易访问，并提供准确清晰的描述是非常重要的。

对于标准UIKiet控件和视图已经有默认的属性信息提供了，对于自定义控件和视图，就要提供必要的信息，具体参见“Crafting Useful Labels and Hints.”

##具体操作

###使用户接口元素易访问

####使自定义的View易访问

如果你的app包含一个自定义的View，你要让这个View易访问，一共有两种方式

第一种:如果是使用IB的，在设置项中把accessibility的选项打开，如果是代码的话，设置为YES


具体代码如下：



	@implementation MyCustomViewController
	- (id)init
	{
  		_view = [[[MyCustomView alloc] initWithFrame:CGRectZero] autorelease];
  		[_view setIsAccessibilityElement:YES];
 
  		/* Set attributes here. */
	}


另一种方式是在你的自定义子类中声明UIAccessibility协议，并实现isAccessibilityElement方法

代码如下：


	@implementation MyCustomView
  	/* Implement attribute methods here. */
 
	- (BOOL)isAccessibilityElement
	{
   		return YES;
	}


####使自定义的容器View中的内容易访问

如果你的app中有包含其他元素的自定义View，当然这些元素是要和用户做交互的，你就必须要让这些元素分别都可以有易访问性。同事还需要确保容器View自己没有易访问性，因为用户交互的是容器View里面的元素，而不是容器本身

要达到这一点，你的自己顶容器View应该实现UIAccessibilityContainer协议，这个协议定义了使容器元素可用的方法

下面代码片段展示了如何实现一个自定义容器View.注意，只有当UIAccessibilityContainer协议调用时，这个容器View创建了易访问元素的数组。获得的结果是，如果iPhone的易访问开关关闭，这个数组就不会创建


	@implementation MultiFacetedView
	- (NSArray *)accessibleElements
	{
   		if ( _accessibleElements != nil )
   		{
      		return _accessibleElements;
   		}
   		_accessibleElements = [[NSMutableArray alloc] init];
 
   		/* 创建一个accessibility元素来表示第一个被包含的元素且作为MultiFacetedView一部分初始化 */
   		UIAccessibilityElement *element1 = [[[UIAccessibilityElement alloc] initWithAccessibilityContainer:self] autorelease];
 
   		/* 设置第一个被包含元素的属性 */
   		[_accessibleElements addObject:element1];
 
   		/* 按照第一个实现第二个 */
   		UIAccessibilityElement *element2 = [[[UIAccessibilityElement alloc] initWithAccessibilityContainer:self] autorelease];
 
   		/* 依葫芦画瓢 */
   		[_accessibleElements addObject:element2];
 
   		return _accessibleElements;
	}
 
	/* 容器本身不应该带易访问性，因此要实现这个方法并返回NO */
	- (BOOL)isAccessibilityElement
	{
   		return NO;
	}
 
	/* 下列方法是UIAccessibilityContainer协议方法的实现 */
	- (NSInteger)accessibilityElementCount
	{
   		return [[self accessibleElements] count];
	}
 
	- (id)accessibilityElementAtIndex:(NSInteger)index
	{
   		return [[self accessibleElements] objectAtIndex:index];
	}
 
	- (NSInteger)indexOfAccessibilityElement:(id)element
	{
   		return [[self accessibleElements] indexOfObject:element];
	}
	@end



###提供准确且有用的属性信息

在提供属性信息的过程中有两个部分：

- 提供简洁，准确且有用的信息
- 确保易访问元素正确的提供内容

####制作有用的Label和Hint
当VoiceOver用户运行你的app的时候，他们一来VoiceOver读出的信息来了解你的app怎么使用，现在就教你如何创建label和hint，以便于让app能够更加简单和友好的为残障者使用。

#####Label创建指南

label属性指明用户界面元素。每一个易访问的用户接口元素，不管是标准的还是自定义的，都应该要提供label的内容。

	注: table的每一行都可以有一个label属性。因此创建table的每行的label属性不同于创建其他控件或者视图的label属性See “Enhance the Accessibility of Table Views” 来学习如何给table的每一行创建label

要确定传什么样的label内容出去，有一个好办法是，把自己代入到盲人的世界，去想象他们用了之后会通过传达的信息指导这是一个怎么样的app。一个好的界面，盲人应该通过读title或者icon知道控件、视图的内容。这是你必须通过label传递出去的信息。

如果提供自定义控件或视图，或者在标准的控件或视图中显示一个自定义图标，则必须提供label属性如下：

- 提供简洁的描述. 理论上来说，label由一个单词组成。例如Add, Play, Delete, Search, Favorites或Volume.有时候一个单词很难表示清楚，也可以用短句，例如“Play music,” “Add name,” or “Add to event.”

- 不要包含控件或视图的类型。类型信息应该存在于traits属性中，不应该在Label中重复。例如，如果你在一个add的按钮的label中包含了控件类型，用户每次访问都会听到"Add button button"的信息，这肯定会让用户感觉很烦躁

- 开头要大写，这可以使VoiceOver用合适的语调读出来
- 不要用句号结尾。label属性不是一个段落，不要用句号结尾
- 文本要本地化. 确保你的app尽可能的本地化，这样用户在设置语言的时候，VoiceOver也会根据设置的语言来朗读你的本地化文本


#####Hint创建指南

hint属性描述了一个控件或视图实施一个行为的结果。当一个行为的结果不能在label中表达的时候理因提供一个hint属性

例如，如果你提供一个播放按钮，当用户按下时，本应该让用户知道按下去的效果是怎么样的。因此，如果你允许用户在按下列表中歌曲名字时播放歌曲，就应该提供一个hint来描述这个结果。因为列表元素中label只描述了元素自己，而不会描述用户点击的结果。

	注: VoiceOver用户可以通过VoiceOver中的选项来确定是否要听hint,默认是要听的
	
如果用户行为不能用label清晰描述，就需要Hint来救场，创建hint的原则如下：

- 提供简洁的描述. 虽然很多控件和视图都需要hint，但是还是要尽可能的短，这样用户听一个的时间不会太长，虽然这么说，但是要避免牺牲清晰度和语法规则，例如把“adds a city”改成“adds city”虽然短了，但是意思不够明确了

- 动词开头，省略主语. 确保使用第三人称来使用动词，如"Plays"而不是祈使句如“Play”.使用第三人称而不是祈使句是为了避免读起来像是命令的口气，这样让用户听起来不舒服。想找到正确的单词，就要想想你是在给你的朋友描述这个控件。你可能会说“Tapping this control plays the song”更多情况下，可能会使用整句中第二段的“Plays the song”来作为hint

- 大写字母开头，句号结尾。虽然一个hint是一个短句而不是一句话，但是用句号结尾能帮助VoiceOver用合适的语气朗读

- 不要在动作和手势中包含名字。一个hint不要告诉用户怎么实施这个行为，而是告诉用户实施这个行为会得到什么结果。因此不要创建例如"Tap to play the song"或者“Tapping purchase the item”或者“Swipe to delete the item ”这样的hint

这是非常重要的，因为VoiceOver用户能够使用VoiceOver指定的手势来和app中元素交互，如果你给不同的手势命名，则可能造成干扰

- 不要在控件或视图中包含名称。用户会从label属性中得到信息，因此你本不应该在hint中再重复信息。不要创建诸如“Save saves your edits”或者“Back returns to the previous screen”的hint

- 不要在控件或视图中包含类型。用户已经通过trait属性知道控件或视图的行为，例如按钮或者搜索框，因此不要创建诸如“Button that adds a name”或“Slider that controls the scale”的hint

- 文本要本地化


####指明合适的Traits

traits属性包含一个或多个单独的trait，然后组合在一起，描述了一个易访问用户接口元素的行为。

	注: 每个独立的traits是用或操作符连接的。

一个标准的UIKit控件，例如一个button或者text field,traits属性是有默认值的.如果只使用标准UIKit控件（没有自定义行为），就不需要改变控件的traits。

如果自定义了标准控件的行为，则需要在原有默认traits的基础上联合一个新的trait。如果创建一个自定义控件或试图，则需要提供元素的traits属性的内容

UI易访问性编程接口定义了12个独立的traits，这里面的很多都是可以连接的。一些traits通过指明伴随着控件（例如按钮）的特殊类型的行为或者元素（例如图片）的类型来描述元素，其他traits通过一个元素可以展现的指定行为来描述一个元素，例如播放声音。

可以使用下列的特征来描述元素：

- Button
- Link
- Search Field
- Keyboard Key
- Static Text
- Image
- Plays Sound
- Selected
- Summary Element
- Updates Frequently
- Not Enabled
- None

通常情况，符合控件的traits是能够成功得通描述行为的traits联合在一起。例如，你可能联合Button的trait同Plays Sound的trait来描述一个自定义控件（这个控件长得像一个Button，当点击时会播放声音）

更多情况，你本应考虑让traits相互更加独立以便于符合特殊的控件，具体的说来，是Button/Link/Search Field和Keyborad Key的traits。你本不应该使用超过1个的traits来描述一个元素。而是应该思考这四个traits中的哪个更加贴近你的需求。如果你的元素有额外的行为，可以再联合一个动作的traits
·

例如，在app里显示一个图片，然后用户点击之后在Safari里打开一个链接。你可以把Image和Link特征结合起来来表述这个事情。另一个例子是键盘上有个键，按下去可以修改其他键，就可以用Keyboard和Selected的特征联合起来表示

可以使用Accessibility Inspector来看标准控件的默认特征值。如何使用Accessibility Inspector，可以参见"Debug Accessibility in iOS Simulator with the Accessibility Inspector"


####增强默认属性信息

标准UIKIt的控件和视图，iOS是提供默认的属性信息的，但是这些信息只是最基本的信息，要想获得好的体验，就要提供自定义信息来增强用户体验。

- 如果使用标准UIKit控件或视图来显示一个系统提供的图标或标题，第一件事，确保你使用的和想要的能够保持一致。然后再决定这个默认label属性是否能精确地表达结果，如果不能，可以考虑提供一个hint属性。例如如果你放置了一个add的按钮在导航栏上，这个按钮是系统提供的UIBarButtonItem类型的add按钮，那么你将得到默认的label属性，值是"添加"，如果用户使用这个按钮的时候总是在添加，则不需要提供额外的hint属性，如果这个按钮可能有歧义，则应该考虑提供一个自定义的hint来描述使用这个空间的结果，例如设置hint的值为“添加一个账号”或者“添加一个评论”
- 如果在标准UIKit的视图上显示一个自定义的图标或者图片，例如一个UIButton，你就需要提供自定义的label属性来描述了。

如果你需要提供或修改accessibility属性，那么可以通过IB直接改(see “Defining Custom Attribute Information in Interface Builder”) 或者通过代码修改 (see “Defining Custom Attribute Information Programmatically”).





###最后：如何打开VoiceOver

设置--》通用--》辅助功能--》VoiceOver 打开即可

###最后的最后：可以用模拟器测试
