---
title: 用JS来理解设计模式（一）
date: 2020-04-28 10:00:00
author: Callas
tags:
- 设计模式
- javaScript
---

# 前言
最近在看Head First设计模式，想好好学一下设计模式相关的知识。这本书要求看书的人需要会java和OO（即面向对象）设计原则，好在两者在学校都学过。但是平时在写js和业务代码的时候很少用到相关的知识，而我们常用到的库和框架，却无处不存在着这些知识，所以想自己写一些库或者框架的人，学会设计模式还是很有必要的。我觉得设计模式和算法有一些相似之处，它们都是一种思想，是一种解决方法。设计模式不是具体的代码和具体的实现，而是和算法一样，是以一种思想存在，当我们想实现某种功能，或者构思框架和库的时候，相关的解决方案便浮现在你的脑海里，所以说设计模式不是说就一定是一种模式，有可能是模式的组合，模式基础上的变化。当然，以上所说也只是我看了前几章的想法。最主要的还是书中所写的实现代码皆是java，所以还是想着用js写一遍，毕竟js看起来不太像是拿来写面向对象的（不过es6已经有Class了）。所以写这个也权当做是自我记录吧。

<!-- more -->

# 策略模式

>**策略模式**定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

看了这个正式定义后可能是不好理解的，不过没关系，讲个书里的例子就明白了。

小李的公司要求他开发一套模拟鸭子的游戏，因此他写了个超类（即父类）Duck，并让各种鸭子继承此超类。（以下代码皆用js来编写）
```
class Duck {
    quack () {} 
    swim () {}
    display () {}
}
```
他给鸭子超类定义了鸭子叫、游泳、展示方法。但是由于每一种鸭子的外观不同，因此在这里的定义是抽象方法（即在超类中不实现，交由子类负责实现，但在js中并没有这种定义），然后在每个鸭子子类中进行实现自己的display方法。

就这样，小李完成了一个简单的模拟鸭子游戏。但是，第二天，产品经理让小李给鸭子加一个飞行的方法。小李心想这还不简单吗，在Duck超类里加一个fly方法就完事了，这就是面向对象的威力。

然而，在第二天的演示中，产品经理惊讶的发现‘橡皮鸭子’也能在屏幕飞来飞去。小李心想糟糕了，于是赶紧在橡皮鸭的类中进行了fly方法**覆盖**，虽然暂时解决了问题，但是一想到日后还要添加各种各样的鸭子，每只️鸭子的飞行方法和叫声都不一样，难道每次添加新类的时候都要去覆盖和检查吗？



> 设计原则： 找出应用中可能需要变化的地方，然后将它们独立出来，不要与那些不需要变化的代码混在一起。

根据这个原则来分析鸭子的行为，就目前来看，鸭子叫和飞行方法是会随着鸭子的不同而改变的，因此需要将两个方法取出来，建立新的类来代表行为。那为什么不取出display呢？这件事书里并没有提到，但是稍稍分析便知道了，display对于每个鸭子子类来说都是必定需要设置的，又不存在相同的情况，因此没有潜在的复用可能性，就是独立成display类也还是依附于特定鸭子子类的，因此没有必要将展示方法取出来。

>设计原则： 针对接口编程，而不是针对实现编程。

这里的接口指的是面向对象里的一个关键字interface，接口就像是一个定义好了所有方法的类，但是其中的每一个方法都不进行实际代码的实现，而是交由继承它的子类来实现方法。它是一种约束，继承了它的子类都必须实现。因此，当我们在使用接口的时候，我们能肯定的去使用继承了某个接口的类的方法，即就是方法必然存在。因此在编写的时候无需关心具体的代码实现（以上是个人的理解）。

因此书中分别建了FlyBehavior和QuackBehavior这两个接口,不过js中并没有接口这种定义，但是js中并没有与接口功能类似的方法存在，只能写个类去代替它，但是这个类并没有约束效力，因此从这里也可以看出来TS的诞生原因。
```
class FlyBehavior {
    fly () {
        console.log('未定义飞行方法');
    }
}

class FlyWithWings extends FlyBehavior {
    fly () {
        console.log('FlyWithWings');
    }
}

class FlyNoWay extends FlyBehavior {
    fly () {
        console.log('FlyNoWay');
    }
}
```

QuackBehavior的实现与上面类似。

>这样的设计，可以让飞行和呱呱叫的动作被其他对象复用，因为这些行为已经与鸭子类无关了。而我们可以新增一些行为，不会影响到既有的行为类，也不会影响使用到飞行行为的鸭子类。

此时再来重新编写鸭子类，就变成了如下。

```
class Duck {
    flyBehavior
    quackBehavior
    swim () {}
    display () {}
    setFly (fly) {
        this.flyBehavior = fly;
    }
    setQuack (quack) {
        this.quackBehavior = quack;
    }
    performFly () {
        this.flyBehavior.fly();
    }
    performQuack () {
        this.quackBehavir.quack();
    }
}
```
可以看到此时我们不再在乎具体的fly方法或者quack 方法是怎么样的，我们只需要去调用方法即可，同时还可以在运行时进行方法的替换，通过调用setQuack与setFly方法。

此时来写一个具体的鸭子类。

```
class MallardDuck extends Duck {
    constructor () {
        super();
        this.setQuack(new Quack());
        this.setFly(new FlyWithWings());
    }
    display () {} // 记得重写
}
```
到这里大概差不多大概就能够理解策略模式是怎么样的了，有时候我们可以把一组行为认为是一族算法，然后封装起来，互相独立，可以随意替换，这差不多就是策略模式的面貌了。
 
> 设计原则：多用组合，少用继承。
