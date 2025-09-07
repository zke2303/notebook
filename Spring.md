

# Spring

## 1、IoC（inversion of Controller）

## Spring IoC 容器和 Bean 简介

### 1、概念

**IoC（Inversion of Control:控制反转）** 是一种设计思想，而不是一个具体的技术实现。IoC 的思想就是将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理。不过， IoC 并非 Spring 特有，在其他语言中也有应用。

### 2、Spring中IoC的实现

在Spring中，通过**DI(dependencies injection：依赖注入)**来实现IoC。

**依赖注入 (DI)** 是 IoC 的一种特殊形式，对象仅通过构造函数参数、工厂方法的参数或在对象实例构造或从工厂方法返回后设置的属性来定义其依赖项（即它们与之协作的其他对象）。然后，IoC 容器在创建 Bean 时注入这些依赖项。这个过程本质上是 Bean 本身的逆过程（因此得名控制反转），Bean 本身通过直接构造类或诸如服务定位器模式之类的机制来控制其依赖项的实例化或位置。

### 3、Bean

在 Spring 中，构成应用程序主干并由 Spring IoC 容器管理的对象称为 Bean。Bean 是由 Spring IoC 容器实例化、组装和管理的对象。



## Container Overview（容器概述）

