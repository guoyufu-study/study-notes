# Head First 设计模式

> https://wickedlysmart.com/head-first-design-patterns/

这本书详解了 14 种设计模式，简略描述了 9 种设计模式。还讲解了组合模式、模式/反模式。

## 概念

### 经验复用

**什么是经验复用？**

经验复用，就是复用自己和其他开发人员的过往的经验与智慧。**复用什么经验？** 解决问题的经验。以前遇到过相同的问题，并已经顺利解决。

**为什么要经验复用？**

知道 OO 基础并不足以让我们设计出良好的 OO 系统。良好的 OO 设计必须具备可复用、可扩展、可维护三个特性。

**如何经验复用？**



### 设计模式

设计模式定义，**什么是设计模式？**

设计模式，是设计层面的模式，是对以往设计经验的系统总结。设计的是“如何组织类和对象以解决某种问题”。

设计模式会告诉我们如何组织类和对象以解决某种问题。



**设计模式与库和框架有什么区别？**

设计模式是设计层面的东西，库和框架是实现层面的东西。设计模式比库和框架的等级更高。

设计模式会告诉我们如何组织类和对象以解决某种问题，而具体如何选择和使用设计模式由程序员做决定，也就是说没有所谓设计模式的库。

库和框架为我们提供某些特定实现，让我们的代码可以轻易地引用。库和框架本身会用到设计模式。

**知道哪些设计模式？**

策略模式、



**为什么要使用设计模式？**

* 便于快速充分沟通，减少误解。

  每个模式都有一个模式名，并且每个模式背后都有一整套它所象征的质量、特性、约束。

  使用模式来沟通，可以让我们用更少的词做更充分的沟通。减少团队成员之间，对彼此的设计看法的误解。

  使用模式谈论软件系统，可以让我们思考架构的层次提高到设计层面，避免让讨论陷入琐碎的实现细节中。

* 方便组织易读、易维护、易扩展的架构







### OO 设计原则

如果行为相同，则需要默认实现，由超类完成声明和实现；如果行为不同，由超类完成声明，由子类完成实现。

#### OO 三原则

封装、继承、多态。

封装，是指封装变化；

继承，目标是代码复用；

多态，是指编程时可以面向接口，而行为会根据实际类型来决定执行。

#### 封装变化

> 很多时候，对代码做局部修改，影响层面不只是局部。比如，在超类上添加方法，会导致所有子类都具备新方法，不应该具备新方法的子类也没办法例外。
>
> 目标：改变系统中的某部分，不会影响到其他部分。

找出应用中可能需要变化的地方，把它们取出并封装起来，以保证改变这一部分不会影响到其他部分。

这样，代码变化引起的不经意后果变少，系统变得更有弹性。

**何时，以及如何定位应用中可能需要变化的地方？**

原则和模式，可以在**软件开发生命周期的任何阶段**应用。

在设计系统时，根据经验，预先考虑到未来可能需要变化的地方，提前在代码中加入这些弹性。

如果每次新的需求一来，都会使某方面的代码发生变化，那么就可以确定，这部分的代码需要被抽取出来，和其他稳定的代码有所区分。



#### 多用组合，少用继承

**为什么要多用组合，少用继承？**

> 第 85 页

利用继承，会在编译时静态决定子类的行为，且所有子类都会继承到相同的行为。

利用组合，可以在运行时动态地扩展对象的行为。

**什么是继承？**

继承，可以实现代码复用。

并不总是能够实现最有弹性和最好维护的设计。

**什么是组合？**

组合（composition），就是将两个类结合起来使用，行为不是继承来的，是和适当的行为对象组合来的。

**为什么要多用组合？**

使用组合建立系统具有很大的弹性：

可以将算法族封装成类，

也可以在运行时动态地改变行为。



#### 面向接口编程

> 面向接口编程，而不是面向实现编程。
>
> 目标：不要把具体实现绑死在类中。

接口，是超类的意思，关键就在多态。

**什么是多态？**

程序可以针对超类型编程，代码执行时会根据对象的实际具体类型执行到真正的行为。

**什么是面向接口编程？**

编写代码时，变量的声明类型应该是超类型，通常是一个抽象类或者是一个接口。只要是具体实现这个超类的类所产生的对象，都可以指定给这个变量。

**面向接口编程有什么好处？**

两个好处：

1. 利用多态调用：不再必须针对具体实现编码调用，可以利用超类型进行多态调用。
2. 避免硬编码实例化动作：不再必须在代码中硬编码子类实例化的动作，可以在运行时才确定具体实现的对象。

示例一：

``` java
// 定义类
interface Animal {
    void makeSound();
}

class Dog implement Animal {
    void makeSound() {
        bark();
    }
    void bark() {
        // 汪汪叫
    }
}

// 针对具体实现编码调用
Dog dog = new Dog();
dog.bark();

// 利用超类型进行多态调用
Animal animal = new Dog();
animal.makeSound();
```

示例二：

``` java
Animal animal = getAnimal();
animal.makeSound();
```



**什么是面向实现编程？**

行为来自超类的具体实现，或者实现某个接口的子类的自行实现。这两种做法都是面向实现编程，我们被实现绑得死死的，除非编写更多代码，否则没办法更改行为。

#### 松耦合

> 努力设计松耦合的交互对象。

为了交互对象之间的松耦合设计而努力。

松耦合的设计能让我们建立有弹性的 OO 系统，能够应对变化。对象之间的互相依赖降到了最低。

#### 开闭原则

> 类应该对扩展开放，对修改关闭。

目标：允许类容易扩展，在不直接修改现有代码的情况下，就可搭配新的行为。比如，装饰者模式。

**为什么不推荐修改现有代码？**

修改现有代码，会增加引进 bug 和产生意外副作用的机会。

**如何让设计的每个部分都遵循开闭原则？**

办不到，也没必要。

遵循开闭原则是有代价的，通常会引入新的抽象层次，增加代码的复杂度。

我们需要把注意力集中在设计中最有可能变化的地方，然后应用开闭原则 。

## 设计模式

### 策略

策略模式定义了算法族，将算法分别封装起来，让他们之间可以互相替换。这个模式让算法的变化独立于使用算法的客户。

#### 个人理解

**封装起来**：封装成类

**互相替换**：可以硬编码替换，也可以运行时动态替换。

**算法的变化**：某个算法内部逻辑变化

**使用算法的客户**：

### 观察者

观察者模式定义了对象之间的一对多依赖。当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。



### 装饰者

装饰者模式动态地给对象添加一些额外的职责。如果要扩展功能，装饰者提供了比继承更有弹性的替代方案。

* 装饰者具有与组件相同的超类型，任何需要使用组件的场合，都可以使用装饰者替代。
* 包装时机不限，包装深度不限
* 装饰者可以给组件添加新行为、扩展新状态。

**继承滥用问题**

类数量爆炸、设计死板、基类加入的新功能并不适用于所有子类。
