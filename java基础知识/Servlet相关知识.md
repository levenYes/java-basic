# Servlet相关知识
## HTTP请求和响应
一个HTTP请求包含三部分内容：方法-URI-协议/版本、请求头信息、请求正文。

一个HTTP响应包含三部分内容：协议-状态码-描述、响应头信息、响应正文。

## Servlet的生命周期
init、service和destroy是生命周期方法。Servlet容器根据以下规则调用这3个方法：

Init，当该Servlet第一次被请求时，Servlet容器会调用这个方法。这个方法在后续请求中不会再被调用。我们可以利用这个方法执行相应初始化工作。调用这个方法时，Servlet容器会传入一个ServletConfig。一般来说，你会将ServletConfig赋给一个类级变量，因此这个对象可以通过Servlet类的其他点来使用。

service，每当请求servlet时，servlet就会调用这个方法。一个线程处理一个请求。

Destroy，当要销毁Servlet时，容器就会调用这个方法。在这个方法里，释放资源。

## Filter是什么
Filter是拦截Request请求的对象：在用户的请求访问资源前处理ServletRequest以及ServletResponse，它用于日志记录、加解密、Session检查等。通过Filter可以拦截处理某个资源或某些资源。

Filter的实现必须继承javax.servlet.Filter接口。这个接口包含了Filter的3个生命周期：init、doFilter、destroy。