# Spring Framework Overview

一些翻译不好的段落就贴上原文了，英文好的朋友有兴趣可以帮忙翻译一下。

Spring使创建Java企业级应用变得简单。它提供了你在企业环境下拥抱Java语言所需要的一切，包括支持Groovy和Kotlin作为JVM上的替代语言，根据应用的需求创建多种架构的灵活性。As of Spring Framework 5.0, Spring requires JDK 8+ (Java SE 8+) and provides out-of-the-box support for JDK 9 already.

Spring支持广泛的应用程序场景。在大型企业中，应用程序经常运行很长时间，并且必须运行在JDK和应用服务器上，而它们的升级周期是不受普通开发者控制的。有的应用则以集成在服务器上的一个单独的jar包的形式运行，可能是在云端环境。还有其他一些可能是不需要服务器的单独运行的应用程序（such as batch or integration workloads）。

1.当我们说Spring的时候我们在谈什么
Spring这个名词在不同的上下文中有不同的含义。它可以用来指Spring Framework这个项目本身，这是它最开始的地方。随着时间的推移，一些其他的Spring项目在Spring Frame的基础之上建立起来，最经常的，人们谈起Spring的时候，是指Spring项目的整个家族。这篇文档中是指他们的基础：Spring Framework。

Spring Framework被分为不同的模块，用户可以选择他们需要用哪个模块。最重要的是核心容器的那些模块，包括一个配置模型系统和依赖注入机制。除此之外，Spring Framework还为不同的应用程序架构提供了基本的支持，包括消息、数据事务、持久化和web。它还包括基于servlet的Spring MVC Web框架和 Spring WebFlux响应式web框架。

A note about modules: Spring’s framework jars allow for deployment to JDK 9’s module path ("Jigsaw"). For use in Jigsaw-enabled applications, the Spring Framework 5 jars come with "Automatic-Module-Name" manifest entries which define stable language-level module names ("spring.core", "spring.context" etc) independent from jar artifact names (the jars follow the same naming pattern with "-" instead of ".", e.g. "spring-core" and "spring-context"). Of course, Spring’s framework jars keep working fine on the classpath on both JDK 8 and 9.

2. Spring和Spring Framework的历史
为了解决早期J2EE规范的复杂性，Spring在2003年出现。虽然有人认为Spring和Java EE是竞争关系，但是Spring实际上是Java EE的补充。Spring编程模型并没有拥抱（遵循？）Java EE的平台规范，而是集成了从企业规范里精心挑选的特有的几个规范。

Servlet API (JSR 340)
WebSocket API (JSR 356)
Concurrency Utilities (JSR 236)
JSON Binding API (JSR 367)
Bean Validation (JSR 303)
JPA (JSR 338)
JMS (JSR 914)
as well as JTA/JCA setups for transaction coordination, if necessary.
Spring Framework还支持依赖注入规范（JSR330）和通用注解规范（JSR250）。应用开发者们可以选择上述方式来代替Spring Framework提供的Spring相关的机制。

对于Spring Framework 5.0，Spring要求至少Java 7，同时在运行时提供了开箱即用的对Java 8的新的API（如Servlet 4.0和 JSON Binding API）的集成。这使得Spring完全兼容Tomcat 8和9，WebSphere 9和JBoss EAP 7。

一直以来，Java EE的角色在应用开发中一直在演变。在Java EE和Spring的早期，应用程序创建后需要部署到应用服务器上。现在，在Spring Boot的帮助下，通过集成Servlet容器（这微不足道），应用程序以一种对开发运维和云端友好的方式创建。就Spring Framework 5而言，一个WebFlux应用甚至不直接使用Servlet API，运行在不是Servlet容器的服务器上（如Netty）。

Spring持续创新和演变。在Spring Framework之外，还有诸如Spring Boot，Spring Security，Spring Data和Spring Cloud以及Spring Batch等其他项目。记住每个项目都有自己的源代码仓库，提交问题栈和发布节奏是非常重要的。

3. 设计哲学
当你学习一个框架的时候，非常重要的一点是，不仅要知道它做了什么，还要知道它所遵循的原则。下面几点是Spring Framework的指导原则。

1.在每一个级别提供选择（个人理解，不强迫用户选择）。Spring让你尽可能晚的推迟设计决定。举个例子，你可以在不更改代码的情况下，切换持久化提供者。这同样适用于许多基础设施（什么鬼）和第三方API。

2.提供多样的视角。Spring拥抱灵活，并且不执拗于事情应该如何做。它从不同的视角支持广泛的应用需求。

3.保持很强的后向兼容性。为了保证版本之间没有阻断性的变化，Spring的演进是精心管理的。Spring支持精心挑选的一系列JDK版本和第三方类库，这使得依赖Spring的应用和类库的维护变得很便利。

4.重视API设计。Spring团队在指定API方面投入了大量的时间和精力，以保证这些API是直观的，并且可以支撑多个版本和持续多年。
