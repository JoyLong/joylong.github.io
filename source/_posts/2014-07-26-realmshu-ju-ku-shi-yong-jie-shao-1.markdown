---
layout: post
title: "Realm数据库使用介绍"
date: 2014-07-26 15:29:11 +0800
comments: true
categories: iOS
---


##1.背景介绍
Realm是新出的移动端数据库，可以运行在手机、平板和可穿戴设备之上，其目标是取代CoreData和SQLite数据库。

Realm由YCombinator孵化的创业公司Realm制作，官网是http://realm.io/

目前支持 iOS 平台，提供 Swift 和 Objective-C的双支持


##2.如何安装
#####传统方式
1、[点击下载最新版Realm](http://static.realm.io/downloads/cocoa/latest)

2、把Realm.framework拖到工程中，添加进去，确保这个framework是存在于你的项目中而不是只链接了引用

3、添加依赖库libc++.dylib

#####CocoaPods方式
1、在podfile中增加pod "Realm"

2、运行`pod install`

3、打开`.xcworkspace`来运行你的项目


##3.使用介绍

###1、Model

Realm的数据Model是RLMObject的子类，添加相关的属性只需要直接用`@property NSString *name;`即可
,和其他NSObject一样，你也可以直接添加一些方法和协议并且直接用。

但是Realm的model是线程限制的。这意味着这个Model只能在创建它的线程中使用

Realm忽略了OC中属性的nonatomic/atomic/strong/copy/weak等，增加属性的时候也不需要加nonatomic之类，如果你加了，那么这些属性特征将会在RLMObject添加到realm时生效

举个例子，假设一个小狗有一个名字，那么这个小狗的Model就是这样

```
// Dog 
@interface Dog : RLMObject

@property NSString *name; 			// 狗的名字

@end

RLM_ARRAY_TYPE(Dog)

```

###2、关系

多个Model之间可以有关系，譬如一对一，一对多，多对多

假设一个小狗不但有自己的名字，每只小狗还有自己的主人，那么就在Model中加入主人，只需要作为property加入即可。

而一个人可以拥有多条小狗，直接在Person的model中添加`@property RLMArray<Dog> *dogs;`

```
// Dog 
@interface Dog : RLMObject

@property NSString *name; 			// 狗的名字
@property NSString *color;			// 狗的毛色
@property Person *owner;			// 狗的主人，一对一

@end

RLM_ARRAY_TYPE(Dog)


// Person
@interface Person : RLMObject 

@property NSString *name;			// 人名
@property RLMArray<Dog> *dogs;		// 拥有的狗，表示可以拥有多条狗
@property RLMArray<House> *houses;	// 拥有的房子

@end

RLM_ARRAY_TYPE(Person)

```

#####具体使用如下
**一对一**
	
	Person *xiaoxin = [[Person alloc] init];
	Dog *dog = [[Dog alloc] init];
	dog.name = @"littlewhite";
	dog.color = @"white";
	dog.owner = xiaoxin;  // 设置小白的主人为小新

**一对多**
	
	// 获取所有毛色中含有白色的小狗
	RLMArray *someDogs = [Dog objectsWhere:@"color contains 'white'"];
	
	xiaoxin.dogs = someDogs;
	[xiaoxin.dogs addObject:dog];
	
注意：RLMArray 属性意味着赋值时复制，property直接赋值将赋值对象的引用。

###3、具体操作
####3.1、增删改如何实现

一个对象的增删改都可以通过`beginWriteTransaction` `commitWriteTransaction`来实现

因为Realm采用的MVCC结构，读事务不会因为写事务正在执行而阻塞

因为model不能在线程中共享，要使用只能先添加到Realm中,只要使用相同的Realm数据库，就可以在多个线程中共享数据

如下：

```
// 创建对象
Person *author = [[Person alloc] init];
author.name    = @"David Foster Wallace";

// 获取数据库
RLMRealm *realm = [RLMRealm defaultRealm];

// 添加到数据库
[realm beginWriteTransaction];
[realm addObject:author];
[realm commitWriteTransaction];
```

####3.2、查询

调用查询语句之后得到的结果是存储这RLMObject数据的RLMArray类型的数组，RLMArray其他特性和NSArray一样

如下，要查询数据库中存储的所有狗，只需要调用Dog这个Model的allObject方法
	
	RLMArray *dogs = [Dog allObjects];

但是上述的查询是在默认数据库中执行，要是想指定一个数据库，就需要

	// 获得指定的数据库	
	RLMRealm *petsRealm = [RLMRealm realmWithPath:@"pets.realm"]; 
	// 从这个数据库中查询所有的小狗
	RLMArray *otherDogs = [Dog allObjectsInRealm:petsRealm];
	

#####3.2.1、条件查询
可以使用`[RLMObject objectsWhere:]`直接写查询条件，也可以使用`NSPredicate`

	// 使用ObjectWhere:做查询条件
	RLMArray *tanDogs = [Dog objectsWhere:@"color = 'tan' AND name BEGINSWITH 'B'"];

	// 使用NSPredicate也有同样的效果
	NSPredicate *pred = [NSPredicate predicateWithFormat:@"color = %@ AND name BEGINSWITH %@", @"tan", @"B"];
	RLMArray *tanDogs2 = [Dog objectsWithPredicate:pred];
	

####3.3、结果排序

结果排序非常简单，使用RLMArray的方法arraySortedByProperty:ascending: 即可

property中输入要排序的字段名


```
// 指定一个排序的字段名，默认是升序
RLMArray *sortedDogs = [[Dog objectsWhere:@"color = 'tan' AND name BEGINSWITH 'B'"]
                    arraySortedByProperty:@"name" ascending:YES];
```

####3.4、链式查询

就是说如果用一些条件已经查询出一个RLMArray数组，还可以继续用这个数组进行条件查询

```
RLMArray *tanDogs = [Dog objectsWhere:@"color = 'tan'"];
RLMArray *tanDogsWithBNames = [tanDogs objectsWhere:@"name BEGINSWITH 'B'"];
```

###4、Realm数据库

默认的Realm数据库可以通过`[RLMRealm defaultRealm]`获得，默认的数据库保存在documents目录下，名字为default.realm， 写事务将自动写入磁盘

####4.1、使用内存中的默认数据库


```
[RLMRealm useInMemoryDefaultRealm]; 
RLMRealm *realm = [RLMRealm defaultRealm]; // 只能在useInMemoryDefaultRealm之后调用
```
这个只对默认数据库有效，而且如果在内存中使用数据库，将无法存储RLMObject,这意味着在app运行中都不能保存数据，但是这个对访问复杂数据很有用，避免了磁盘开销

####4.2、其他数据库

还可以使用`[RLMRealm realmWithPath:]` `[RLMRealm realmWithPath:readOnly:error:]`来获得别的数据库

####4.3、线程间使用Realm数据库

要在多个线程间访问同一个数据库，可以用`[RLMRealm defaultRealm]` `[RLMRealm realmWithPath:]` 或者 `[RLMRealm realmWithPath:readOnly:error:]`在每一个线程中获得相应的数据库对象。只要地址用的对，数据库肯定是同一个，但是`不要在线程间共享RLMRealm对象`。有多个Realm对象指向同样的文件地址，这个是没有问题的，甚至可以同一个线程中有多个Realm对象指向同一个文件地址，也是安全的。



###5、通知
当Realm更新的时候会发出通知，可以注册一个block 来响应通知
	
	self.token = [realm addNotificationBlock:^(NSString *note, RLMRealm *realm) {
		[muViewController updateUI];
	}];

还可以用`[Realm removeNotificationBlock:]`移除block

###6、后台操作
Realm在大数据写入上非常有效率。事务可以使用GCD在后台实施，从而避免阻塞主线程。RLMRealm不是县城安全的，也不能在线程中共享，因此当需要读写数据库时，必须在每个线程或调度队列里获取一个RLMRealm实例。

下面是一个后台插入百万级数据的例子

	
	dispatch_async(queue, ^{
		RLMRealm *realm = [RLMRealm defaultRealm];
		
		for (NSInteger idx1 = 0; idx1 < 1000; idx1++) {
			[realm beginWriteTransaction];
			for (NSInteger idx2 = 0; idx2 < 1000; idx2++) {
				[Person createInRealm:realm
						withObject:@{@"name : [self randomString],
									 @"birthdate" : [self randomDate]}];
			}
			[realm commitWriteTransaction];
		}
	});
	


