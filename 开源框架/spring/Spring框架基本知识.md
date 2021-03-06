# Spring框架基本知识
## 关于Spring IoC
Spring IoC包含了最为基本的IoC容器BeanFactory的接口与实现，也就是说，在这个Spring的核心包中，不仅定义了IoC容器的最基本接口BeanFactory，也提供了一系列这个接口的实现，如XmlBeanFactory就是一个最基本的BeanFactory（IoC容器），从名字上可以看到，它能够支持通过XML文件配置的Bean定义信息。

另外，在BeanFactory接口实现中，Spring还设计了IoC容器的高级形态ApplicationContext应用上下文供用户使用，如FileSystemXml\~，对应用来说，是IoC容器中更面向框架的使用方式。

## BeanFactory接口
作为IoC容器，也需要为它的具体实现指定基本的功能规范，表现为接口类BeanFactory，它体现了Spring为提供给用户使用的IoC容器所设定的最基本的功能规范。

以XmlBeanFactory为例，它继承自DefaultListableBeanFactory这个类，实际上已经包含了IoC容器所具有的重要功能。后者则是继承自AbstractAutowireCapableBeanFactory，再上去就是继承自AbstractBeanFactory。

Spring通过定义BeanDefinition来管理基于Spring的应用中的各种对象以及它们之间的相互依赖关系。BeanDefinition抽象了我们对Bean的定义，是让容器起作用的主要数据类型。

弄清楚BeanFactory和ApplicationContext之间的区别和联系，意味着我们具备了辨别容器系列中不同容器产品的能力。还有一个好处就是，如果需要定制特定功能的容器实现，也能比较方便地在容器系列中找到一款恰当的产品作为参考，不需要重新设计。

## 使用IoC容器的步骤
1）创建IoC配置文件的抽象资源，这个抽象资源包含了BeanDefinition的定义信息。
2）创建一个BeanFactory，这里使用DefaultListableBeanFactory。
3）创建一个载入BeanDefinition的读取器，这里使用XmlBeanDefinitionReader来载入XML文件形式的BeanDefinition，通过一个回调配置给BeanFactory。
4）从定义好的资源位置读入配置信息，具体的解析过程由XmlBeanDefinitionReader来完成。完成整个载入和注册Bean定义之后，需要的IoC容器就建立起来了。这时候就可以使用IoC容器了。

## ApplicationContext容器的设计原理
在FileSystemXmlApplicationContext的设计中，我们看到ApplicationContext应用上下文的主要功能已经在它的基类AbstractXmlApplicationContext中实现了。作为一个具体的上下文，只需要实现和它自身设计相关的两个功能。

一个功能是，如果应用直接使用FileSystemXmlApplicationContext，对于实例化这个应用上下文的支持，同时启动IoC容器的refresh()过程。

另一个功能与怎样从文件系统中加载XML的Bean定义资源有关。

## IoC容器的初始化过程
简单来说，IoC容器的初始化是由前面介绍的refresh()方法来启动的，这个方法标志着IoC容器的正式启动。具体来说，这个启动包括BeanDefinition的Resource定位、载入和注册三个基本过程。

Spring把这个三个过程分开，并使用不同的模块来完成，如使用相应的ResourceLoader、BeanDefinitionReader等模块，通过这样的设计方式，可以让用户更加灵活地对这三个过程进行剪裁或扩展，定义出最适合自己的IoC容器的初始化过程。

第一个过程是Resource定位过程。指的是BeanDefinition的资源定位，它是由ResourceLoader通过统一的Resource接口来完成，这个Resource对各种形式的BeanDefinition的使用都提供了统一接口。

第二个过程是BeanDefinition的载入。这个载入过程是把用户定义好的Bean表示成IoC容器内部的数据结构，而这个容器内部的数据结构就是BeanDefinition，实际上就是POJO对象在IoC容器中的抽象，方便管理。

第三个过程是BeanDefinition的注册。调用BeanDefinitionRegistry接口的实现，把载入过程得到的BeanDefinition注入到一个HashMap中去。

Bean定义的载入和依赖注入是两个独立的过程。依赖注入一般发生在应用第一次通过getBean向容器索取Bean的时候。但有一个例外值得注意，如果我们对某个Bean设置了lazyinit属性，那么在容器初始化时就会预先注入。

## 详解IoC容器的初始化过程
在BeanDefinition定位完成的基础上，就可以通过返回的Resource对象来进行BeanDefinition的载入了。在定位过程完成之后，为BeanDefinition的载入创造了I/O操作的条件，但是具体的数据还没有读入。

在IoC容器中建立了对应的数据结构，或者说可以看成是POJO对象在IoC容器中的抽象，这些数据结构可以以AbstractBeanDefinition为入口，让IoC容器执行索引、查询和操作。

这个注册为IoC容器提供了更友好的使用方式，在DefaultListableBeanFactory中，是通过一个HashMap来持有载入的BeanDefinition的。将解析得到的BeanDefinition向IoC容器中的beanDefinitionMap注册的过程是在载入BeanDefinition完成后进行的。（可画图）

总结来说，这个初始化过程完成的主要工作是在IoC容器中建立BeanDefinition数据映射。（没有涉及依赖注入）

因为在ApplicationContext中，Spring已经为我们提供了一系列加载不同Resource的读取器的实现，而DefaultListableBeanFactory只是一个纯粹的IoC容器，需要为它配置特定的读取器才能完成这些功能。如果使用这种更底层的容器，能提高制定IoC容器的灵活性。

## 依赖注入
getBean是依赖注入的起点，之后会调用createBean，下面通过createBean代码来了解这个实现过程。在这个过程中，Bean对象会依据BeanDefinition定义的要求生成。在AbstractAutowireCapableBeanFactory中实现了这个createBean，createBean不但生成了需要的Bean，还对Bean的初始化进行了处理，比如实现了在BeanDefinition中的init-method属性定义，Bean后置处理器等。

与依赖注入关系特别密切的方法有createBeanInstance和populateBean，下面分别介绍这两个方法。在createBeanInstance中生成了Bean所包含的Java对象，这个对象的生成有很多种不同的方式，可以通过工厂方法生成，也可以通过容器的autowire特性生成，这些生成方式都是由BeanDefinition来指定的。

在IoC容器中，要了解怎样使用CGLIB来生成Bean对象，需要看一下SimpleInstantiationStrategy类。这个Strategy是Spring用来生成Bean对象的默认类，它提供了两种实例化Java对象的方法，一种通过Utils，它使用了JVM的反射功能，一种是通过前面提到的GCLIB来生成。

Spring是怎样对这些对象进行处理的，也就是bean对象生成以后，怎样把这些Bean对象的依赖关系设置好，完成整个依赖注入过程。整个过程涉及对各种Bean对象的属性的处理过程（即依赖关系处理的过程），这些依赖关系处理的依据就是已经解析得到的BeanDefinition。

在Bean的创建和对象依赖注入的过程中，需要依据BeanDefinition中的信息来递归地完成依赖注入。从上面的几个递归过程中可以看到，这些递归都是以getBean为入口的。一个递归是在上下文体系中查找需要的Bean和创建Bean的递归调用；另一个递归是在依赖注入时，通过递归调用容器的getBean方法，得到当前Bean的依赖Bean，同时也触发对依赖Bean的创建和注入。在对Bean的属性进行依赖注入时，解析的过程也是一个递归地过程。这样，依据依赖关系，一层一层地完成Bean的创建和注入，直到最后完成当前Bean的创建。有了这个顶层Bean的创建和它的属性依赖注入的完成，意味着和当前Bean相关的整个依赖链的注入也完成了。

在Bean创建和依赖注入完成以后，在IoC容器中建立起一系列依靠依赖关系联系起来的Bean，这个Bean已经不是简单的Java对象了。该Bean系列以及Bean之间的依赖关系建立完成以后，通过IoC容器的相关接口方法，就可以非常方便地供上层应用使用了。

（1）DefaultListableBeanFactory调用AbstractBeanFactory.doGetBean()
（2）AbstractAutowireCapableBeanFactory.createBean()
（3）SimpleInstantiationStrategy.instantiate()
（4）SimpleInstantiationStrategy.populateBean()
（5）AbstractAutowireCapableBeanFactory.applyPropertyValues
（6）BeanDefinitionResolver. resolveReference()

## Spring MVC的实现
#### IoC容器启动的基本过程
IoC容器的启动过程就是建立上下文的过程，该上下文是与ServletContext相伴而生的，同时也是IoC容器在Web应用环境中的具体表现之一。由ContextLoaderListener启动的上下文为根上下文。在根上下文的基础上，还有一个与Web MVC相关的上下文用来保存控制器（DispatcherServlet）需要的MVC对象，作为根上下文的子上下文，构成一个层次化的上下文体系。

在Web容器中启动Spring应用程序时，首先建立根上下文，然后建立这个上下文体系的，这个上下文体系的建立是由ContextLoader来完成的。
#### SpringMVC实现的大致步骤
1）建立Controller和HTTP请求之间的映射关系。这个工作是由在handlerMapping中封装的HandlerExecutionChain对象来完成的，而对Controller控制器和HTTP请求的映射关系的配置是在Bean定义中描述，并在IoC容器初始化时，通过初始化HandlerMapping中完成的，这些定义的关系会被载入到一个handlerMap中使用。

2）在MVC框架接收到HTTP请求的时候，DispatcherServlet会根据具体的URL请求信息，在HandlerMapping中进行查询，从而得到对应的HandlerExecutionChain，在里面封装了配置的Controller，这个请求对应的Controller会完成请求的相应动作，生成需要的ModelAndView对象，这个对象就像它的名字所表示的一样，可以从该对象中获得Model模型数据和视图对象。

3）DispatcherServlet把获得的模型数据交给特定的视图对象，从而完成这些数据的视图呈现工作。这个视图呈现由视图对象的render方法完成。对应于不同的视图对象，render方法会完成不同的视图呈现处理。

## Spring AOP设计原理
在Spring的AOP模块中，一个主要的部分是代理对象的生成，而对于Spring应用，可以看到，是通过配置和调用Spring的ProxyFactoryBean来完成这个任务的。在ProxyFactoryBean中，封装了主要代理对象的生成过程。在这个生成过程中，可以使用JDK的Proxy和CGLIB两种生成方式。












