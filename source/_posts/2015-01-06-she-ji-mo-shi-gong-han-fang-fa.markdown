---
layout: post
title: "设计模式-工厂方法"
date: 2015-01-06 13:37:53 +0800
comments: true
categories: 设计模式
---

##什么是工厂方法

<p style="color:#666666;font-weight:bold;font-size:18px;line-height: 130%;text-indent:20pt">定义创建对象的接口，让子类决定实例化哪一个对象。工厂方法使得一个类的初始化延迟到其子类。工厂方法也被称为虚构造器。</p>
<div style="text-align:center">
<img src = "/images/工厂方法类图.png" style = "border:1px dashed #930f83;align:center"/>
</div>
<p style = "text-indent:20pt;">抽象的Product定义了工厂方法创建的对象接口。ConcreteProduct实现了Product定义的接口。Creator定义了一个工厂方法，这个方法会返回一个Product对象，当然也可以为这个工厂方法提供一个默认的实现，返回一个默认的ConcreteProduct对象。Creator的其他方法可以调用这个工厂方法创建Product对象。ConcreteCreator是Creator的子类，它重载了工厂方法，并返回一个ConcreteProduct的实例。

##何时使用工厂方法

<p>当遇到以下情形时，可以考虑使用工厂模式
<ol>
<li>编译时无法准确预期想要创建的对象的类；
<li>类想让其子类决定在运行时创建什么；
<li>类有若干辅助类为其子类，而你想将返回哪个类这一信息局部化
</ol>

####Cocoa中的例子
<p style = "text-indent:20pt;">Cocoa中使用这一架构的例子有NSNumber。尽管可以使用常见的alloc init两步法创建NSNumber的实例，但这基本上没做什么，除非你使用预先定义的类工厂方法来创建一个有意义的实例。例如，
{% codeblock lang:objc %}
[NSNumber numberWithBool:YES]
{% endcodeblock%}这个方法会得到一个NSCFBoolean的实例，这个实例包含传给类工厂方法的布尔值。	
##使用工厂方法的效果

<p style="text-indent:20pt">工厂方法不再将与特定应用有关的类绑定到你的代码中，代码仅处理Product接口；因此它可以与用户定义的任何ConcreteProduct类一起使用。
<p style="text-indent:20pt">工厂方法的一个潜在缺点在于用户仅仅为了创建一个特定的ConcreteProduct对象，就不得不创建Creator的子类。当创建Creator子类不必需时，客户现在必然要处理类演化的其他方面；但是当客户无论如何必须创建Creator的子类时，创建子类也是可行的。

####下面是Factory Method模式的另外两种效果
<ol>
<li>为子类提供挂钩（hook）</li>
用工厂方法在一个类的内部创建对象通常比直接创建对象更灵活。Factory Method给子类一个挂钩以提供对象的扩展版本。
Creator可以定义工厂方法的合理的缺省实现。
<li>连接平行的类层次</li>
当一个类将它的职责委托给一个独立的类的时候，就产生了平行类层次。
工厂方法将哪些类应一同工作的信息局部化了。
</ol>

##工厂方法的实现

<p style = "text-indent:20pt;">当应用Factory Method模式时，应考虑一下问题
<ol>
	<li>主要有两种不同的情况
		<ul>
			<li>Creator是一个抽象类，并且不提供它所声明的工厂方法的实现。这种情况需要子类来定义实现。它避免了不得不实例化不可预见类的问题
			<li>Creator是一个具体的类，而且为工厂方法提供一个缺省的实现。它所遵循的准则是：用一个独立的操作创建对象，这样子类才能重新定义他们的创建方式。
		</ul>
	</li>
	<li>参数化工厂方法</li>
	工厂方法采用一个标识要背创建的对象种类的参数。
</ol>

##典型应用[来自网络]

<p style = "text-indent:20pt;">要说明工厂模式的优点，可能没有比组装汽车更合适的例子了。场景是这样的：汽车由发动机、轮、底盘组成，现在需要组装一辆车交给调用者。假如不使用工厂模式，代码如下：
``` objc
@implementation Engine

+ (instancetype)getEngine
{
    Engine *engine = [Engine new];
    NSLog(@"这是汽车的发动机");
    return engine;
}

@end

@implementation Underpan

+ (instancetype)getUnderpan
{
    Underpan *underpan = [Underpan new];
    NSLog(@"这是汽车的底盘");
    return underpan;
}

@end

@implementation Wheel

+ (instancetype)getWheel
{
    Wheel *wheel = [Wheel new];
    NSLog(@"这是汽车的轮胎");
    return wheel;
}

@end


@implementation Car

+ (instancetype)makeCarWithEngine:(Engine *)engine underpan:(Underpan *)underpan wheel:(Wheel *)wheel
{
    Car *car = [Car new];
    return car;
}

@end

@implementation Client

- (Car *)mainMethod
{
    Engine *engine = [Engine getEngine];
    Underpan *underpan = [Underpan getUnderpan];
    Wheel *wheel = [Wheel getWheel];
    Car *car = [Car makeCarWithEngine:engine underpan:underpan wheel:wheel];
    return car;
}
@end
```

<p style="text-indent:20pt">可以看到，调用者为了组装汽车还需要另外实例化发动机、底盘和轮胎，而这些汽车的组件是与调用者无关的，严重违反了迪米特法则，耦合度太高。并且非常不利于扩展。另外，本例中发动机、底盘和轮胎还是比较具体的，在实际应用中，可能这些产品的组件也都是抽象的，调用者根本不知道怎样组装产品。假如使用工厂方法的话，整个架构就显得清晰了许多。

{% codeblock lang:objc %}
@interface Factory : NSObject
+ (Car *)makeCar;
@end

@implementation Factory

+ (Car *)makeCar
{
    Engine *engine = [Engine getEngine];
    Underpan *underpan = [Underpan getUnderpan];
    Wheel *wheel = [Wheel getWheel];
    Car *car = [Car makeCarWithEngine:engine underpan:underpan wheel:wheel];
    return car;
}

@end

@implementation Client

- (Car *)mainMethod
{
    Car *car = [Factory makeCar];
    return car;
}
@end

{% endcodeblock %}

<p style="text-indent:20pt">  使用工厂方法后，调用端的耦合度大大降低了。并且对于工厂来说，是可以扩展的，以后如果想组装其他的汽车，只需要再增加一个工厂类的实现就可以。无论是灵活性还是稳定性都得到了极大的提高。