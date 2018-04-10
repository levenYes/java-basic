# Spring MVC相关知识
## 1、跟踪Spring MVC的请求
DispatcherServlet就是前端控制器。DispatcherServlet的任务是将请求发送给controller。它会查询一个或多个处理器映射（handler mapping）来确定请求的下一站在哪里。处理器映射会根据请求所携带的URL信息来进行决策。

实际上，设计良好的控制器本身只处理很少甚至不处理工作，而是将任务逻辑委托给一个或多个服务对象进行处理。

控制器在完成逻辑处理后会产生信息，需要返回给用户并在浏览器上显示。这些信息称之为MODEL。信息需要发送给一个视图（view），通常会是JSP。

控制器所做的最后一件事就是将模型数据打包，并且标识出用于渲染输出的视图名。它接下来会将请求连同模型和视图名发送回DispatcherServlet。

这样控制器就不会与特定的视图相耦合，传递给DispatcherServlet的视图名仅仅只是一个逻辑名称。DispatcherServlet将会使用视图解析器（view resolver）来将逻辑视图名匹配为一个特定的视图实现。
## 2、两个应用上下文之间的故事
当DispatcherServlet启动的时候，它会创建Spring应用上下文，并加载配置文件或配置类中所声明的bean。

DispatcherServlet加载包含web组件的bean，如控制器、视图解析器以及处理器映射，而ContextLoaderListener要加载应用中的其他bean。这些bean通常是驱动应用后端的中间层和数据层组件。

## 3、接受请求的输入
Spring MVC允许以多种方式将客户端中的数据传送到控制器的处理器方法中，包括：查询参数（Query Parameter）、表单参数（Form Parameter）、路径变量（Path Variable）。类似@RequestParam(“id”)

## 4、理解视图解析
如果控制器只通过逻辑视图名来了解视图的话，那Spring该如何确定使用哪一个视图实现来渲染模型呢？这就是Spring视图解析器的任务了。(FreeMarkerViewResolver)

View接口的任务就是接受模型以及Servlet的request和response对象，并将输出结果渲染到response中。

## 5、跨重定向请求传递数据
在处理完POST请求后，通常来讲一个最佳实践就是执行以下重定向。除了其他的一些因素外，这样做能够防止用户点击浏览器的刷新按钮或后退箭头时，客户端重新执行危险的POST请求。

在控制器方法返回的视图名称中，我们借助了“redirect：”前缀的力量。当控制器方法返回的String值以“redire：”开头的话，那么这个String不是用来查找视图的，而是用来指导浏览器进行重定向的路径。
