---
title: 浅析Field-injection-is-not-recommended
date: 2023-02-22 17:36:46
tags:
	- field injection
categories:
	- 编程语言
---

IDEA运行SpringBoot项目，遇到以下有关 @Autowired 注解的警告：Field injection is not recommended . 这篇文章浅析这个问题，为什么会有这样的提示？为什么字段注入的方式不推荐？

<!-- more -->

当前的 spring framework (5.0.3) 文档仅定义了两种主要的注入类型[^1]

> DI exists in two major variants: Constructor-based dependency injection and Setter-based dependency injection.

基于构造函数的依赖注入:在基于构造函数的依赖注入中，类构造函数被注释@Autowired并包含可变数量的参数以及要注入的对象。

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private final MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}

```
基于构造函数的注入的主要优点是您可以将注入的字段声明为final，因为它们将在类实例化期间启动。这对于所需的依赖项很方便。

基于Setter的依赖注入:在基于 setter 的依赖注入中，setter 方法用@Autowired. 一旦使用无参数构造函数或无参数静态工厂方法实例化 Bean，Spring 容器将调用这些 setter 方法以注入 Bean 的依赖项。

```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}

```

但实际上还有第三种，也是被广泛应用的

基于字段的依赖注入:在基于字段的依赖注入中，字段/属性用@Autowired. 实例化类后，Spring 容器将设置这些字段。

```java
@Component
public class ConstructorBasedInjection {

    @Autowired
    private InjectedBean injectedBean;

}
```

这是注入依赖项的最简洁的方法，因为它避免了添加样板代码，并且无需为类声明构造函数。代码看起来不错，简洁明了，但正如代码检查员已经提示我们的那样，这种方法有一些缺点。

那么为什么不推荐使用基于字段的依赖注入？ 

## 基于字段的依赖注入缺点[^2]

### 不允许不可变字段声明

基于字段的依赖注入不适用于声明为 final/immutable 的字段，因为这些字段必须在类实例化时实例化。声明不可变依赖项的唯一方法是使用基于构造函数的依赖项注入。

### 违反了单一责任原则

如您所知，在面向对象的计算机编程中，[SOLID](https://en.wikipedia.org/wiki/SOLID)[^3]首字母缩略词定义了五个设计原则，这些原则将使您的代码易于理解、灵活和可维护。SOLID中的S代表单一职责原则，这意味着一个类应该只负责软件应用程序功能的单个部分，并且它的所有服务都应该与该职责严格对齐。

使用基于字段的依赖注入，很容易在你的类中有很多依赖，一切看起来都很好。如果改为使用基于构造函数的依赖注入，随着更多的依赖项被添加到你的类中，构造函数变得越来越大。拥有一个包含十个以上参数的构造函数是一个明显的标志，表明该类有太多协作者，这可能是开始将类拆分为更小且更易于维护的部分的好时机。 

> 这里要说明：基于构造函数依赖注入，并不说能够解决类里面的过多依赖的问题。而是说能够直观的提示我们：这个类被注入了太多的依赖，你该停下来，优化并拆分你的业务逻辑了！

因此，尽管字段注入并不直接导致打破单一责任原则，但它肯定有助于隐藏信号，否则这些信号会非常明显。

### 与依赖注入容器紧密耦合

使用基于字段的注入的主要原因是避免getter和setter的样板代码或为您的类创建构造函数。最后，这意味着可以设置这些字段的唯一方法是通过Spring容器实例化类并使用反射注入它们，否则字段将保持为 null 并且您的类将被破坏/无用。

依赖注入设计模式将类依赖的创建与类本身分开，将此责任转移到类注入器，允许程序设计松散耦合并遵循单一职责和依赖倒置原则（再次是SOLID）。因此，最终通过自动装配其字段实现的类解耦会因再次与类注入器（在本例中为 Spring）耦合而丢失，从而使该类在 Spring 容器之外无用。

这意味着如果你想在应用程序容器之外使用你的类，例如用于单元测试，你必须使用 Spring 容器来实例化你的类，因为没有其他可能的方法（除了反射）来设置自动装配的字段。

### 隐藏的依赖

使用依赖注入模式时，受影响的类应该使用公共接口清楚地公开这些依赖关系，方法是在构造函数中公开所需的依赖关系，或者使用方法（setter）公开可选的依赖关系。当使用基于字段的依赖注入时，该类本质上将这些依赖隐藏到外部世界。

### 结论
我们已经看到应该尽可能避免基于字段的注入，因为它有许多缺点，无论它看起来多么优雅。推荐的方法是使用基于构造函数和基于设置器的依赖注入。对于必需的依赖项，建议使用基于构造函数的注入，以允许它们不可变并防止它们为空。对于可选的依赖项，建议使用基于 Setter 的注入。


---

补充 2023-02-27

1. 构造器依赖注入，如果要注入的属性太多，构造方法会很臃肿，可以在类上加 `@RequiredArgsConstructor` 注解，这个注解会把final修饰的（或者@NonNull注解的）属性构建默认的构造方法。

```java
@RequiredArgsConstructor
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private final MovieFinder movieFinder;

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

2. 从Spring Framework 4.3开始，如果目标bean只定义了一个构造函数，则不再需要在这样的构造函数上使用@Autowired注释。但是，如果有几个可用的构造函数，至少必须用@Autowired注释其中一个，以便指示容器使用哪个构造函数[^4]。


[^1]:[《Spring Framework Documentation 1.4.1. Dependency Injection》](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators)
[^2]:[《Field injection is not recommended – Spring IOC》](https://blog.marcnuri.com/field-injection-is-not-recommended#eases-single-responsibility-principle-violation)
[^3]:[《Wiki SOLID》](https://en.wikipedia.org/wiki/SOLID)
[^4]:[《Core Technology:1.9.2. Using @Autowired》](https://docs.spring.io/spring-framework/docs/5.2.3.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation)