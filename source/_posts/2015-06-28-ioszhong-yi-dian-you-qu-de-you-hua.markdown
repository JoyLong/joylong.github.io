---
layout: post
title: "iOS中一点有趣的优化"
date: 2015-06-28 20:53:38 +0800
comments: true
categories: iOS
---

###iOS中一点有趣的优化

声明四个数组

```
NSArray *arr1 = [[NSArray alloc] init];

NSArray *arr2 = [[NSArray alloc] init];

NSArray *arr3 = [[NSArray alloc] init];

NSArray *arr3 = [[NSArray array];

NSLog(@"arr1 = %p, arr2 = %p, arr3 = %p, arr4 = %p", arr1, arr2, arr3 , arr4);

NSLog(@"arr1 = %p, arr2 = %p, arr3 = %p, arr4 = %p", &arr1, &arr2, &arr3 , &arr4);
```

第一个NSLog打印出来的结果是


`1、0x7fa868401cd0 `
`2、0x7fa868401cd0 `
`3、0x7fa868401cd0`
`4、0x7fa868401cd0`

第二个NSLog打印出来的结果是

`1、0x7fff5d1aab18 `
`2、0x7fff5d1aab10 `
`3、0x7fff5d1aab08 `
`4、0x7fff5d1aab00`

会发现，为什么第一个地址是完全一样的，而第二个地址各自不一样

`这是苹果系统中的一个优化：因为[[NSArray alloc] init]和[NSArray array]都是初始化一个不可变数组，这个数组为空，因此苹果将空的不可变数组做了优化，开辟了一个空间，所有的不可变数组都指向这里，第二个NSLog是打印的arr1 arr2 arr3 arr4的地址，这表示的是这几个变量分配的内存地址，而不是NSArray分配的地址`

`如果将NSArray改成NSMutableArray，然后打印出来，则会发现四个地址又都不一样了，这是因为不可变数组的内容可能有多个，因此每一个都开辟了新的空间`