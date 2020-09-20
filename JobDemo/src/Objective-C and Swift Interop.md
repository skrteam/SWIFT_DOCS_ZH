# OC和Swift是如何交互的 Objective-C and Swift Interop

```
原标题：Objective-C and Swift Interop
地址：https://github.com/apple/swift/blob/master/docs/ObjCInterop.md
```
本文用于描述Swift是如何与OC代码和OC运行时交互的。

Swift运行时和OC运行时之间的交互是体现在实现细节上的，并且是可能发生变化的）！不要在Swift运行时之外编写依赖这些细节的代码。

当前文档仅适用于支持OC交互的平台。至于其他平台，Swift会把OC相关内容略去。

## 消息发送

Swift会像OC编译器一样生成`objc_msgSend`函数的调用和变量来发送一个OC的消息。被标记或推断为@objc类型的Swift方法会被暴露在OC类的方法列表里。

## 类
Swift类兼容OC类。当Swift类继承自OC类，它会像OC类的继承方式一样，通过生成一个OC类的结构体并且`superclass`的指针指向其对应的父类。在运行时，没有父类的纯净Swift类会生成一个OC类，它是内部`SwiftObject`类的子类。`SwiftObject`实现了最小接口，允许这些对象在具备正确内存管理和基本功能的情况下通过OC代码传递。

## 编译器生成的类
Swift类就像OC的类一样，可以生成为二进制静态数据的一部分。编译器提供了一个与OC类结构匹配的结构，该结构额外添加了Swift特定的域（fileds）。

## 动态生成的类
有些Swift类必须在运行时生成，如泛型类。对于这些类，Swift运行时使用`MetadataAllocator`来为其分配空间，并用适当的类结构填充，最后使用**SPI** （ System Programming Interfaces）`objc_readClassPair`向OC运行时注册新类。

## Stub类
注：Stub类只在 macOS 10.15+, iOS/tvOS 13+, 和 watchOS 6+以上支持。在旧的OS系列系统的OC运行时上不存在必要的调用来支持Stub类。Swift编译器仅在针对支持它们的OS版本时才会产生Stub类。

Stub类可以为动态生成的类生成，这些（dynamically-generated）类在编译时已知，但其大小在运行时才能知道。因为类的大小是未知的，编译器不知道要保留多少空间，因此编译器不能静态地设置它们。取而代之的是，编译器会生成一个`stub class`。Stub类包括：

``` c
// uintptr_t 一个能够存储指针的无符号int
uintptr_t dummy;// 虚拟变量
uintptr_t one;
SwiftMetadataInitializer initializer;//Swift 元类初始化器
```

`dummy`域的存在是用于欺骗链接器的（以链接器免链接不到真实地址而报错）。Stub类的符号指向one域，并且使用`.alt_entry`指令来标明它与`dummy`域相关联。

`one`字段像普通OC类的`isa`字段一样。OC运行时根据这个字段来区分Stub类和普通类。介于1和15之间（含1和15）的值表示Stub类。 当前保留非1的值(以用于兼容后面可能出现的其他的情况)。

OC编译器可以从OC代码引用这些类。对Stub类的引用必须经过`classref`，它是指向类指针的指针。`classref`中类指针的低标志位标识是否需要初始化它。当低位被设置，OC运行时将其视为指向Stub类的指针，并使用初始化函数来检索真实的类。当低标志位被清除时，OC运行时将其视为指向实际类的指针，而不执行任何操作。

然后，编译器必须通过生成对OC运行时函数`objc_loadClassref`的调用来访问该类，该调用返回已初始化并已重定位的类。 例如，如下代码：
``` c
[SwiftStubClass class]
``` 

生成类似如下的代码：
``` c
static Class *SwiftStubClassRef =
  (uintptr_t *)&_OBJC_CLASS_$_SwiftStubClassRef + 1;
Class SwiftStubClass = objc_loadClassref(&SwiftStubClassRef);
objc_msgSend(SwiftStubClass, @selector(class));
```

初始化函数负责设置新类并将其返回给OC运行时。 它必须是幂等的：OC运行时不采取任何预防措施以避免在多线程环境中多次调用初始化程序，并希望初始化程序函数本身可以满足任何此类需求。

初始化函数需要通过在OC运行时调用`_objc_realizeClassFromSwift` **SPI**来出则新生成的类。它必须提供一个新类和Stub类。 OC运行时使用此信息来建立从Stub类到真实类的映射。 这种映射允许Stub类上的OC类别起以下作用：从Swift实现的Stub类，任何与该Stub类相关联的类别都将添加到相应的真实类中。

## 记录
该文件不完整。 它应该扩展为包括：
- 有关ObjC和Swift类结构的ABI的信息。
- is-Swift位。
- 旧版与稳定的ABI和is-Swift位重写。
- Swift运行时使用的Objective-C运行时挂钩。
- 更多？