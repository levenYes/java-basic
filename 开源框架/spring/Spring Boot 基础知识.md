# Spring Boot 基础知识
## Spring的缺点
1.XML配置已经不是流行的系统配置方式
2.集成第三方工具时，需要考虑工具之间的兼容性
3.系统启动慢，不具备热部署功能，完全依赖虚拟机或者web服务器的热部署

## Spring Boot的优点
1.约定大于配置
2.提供了内置的tomcat或jetty容器
3.通过依赖的jar包管理，自动装配技术（starter），容易支持第三方工具
4.支持热加载和系统监控

## 使用热部署
Spring Boot提供了spring-boot-devtools，它能在修改类或者配置文件的时候自动重新加载Spring Boot应用。需要打开pom文件，添加相应依赖。

修改某一个方法后，观察控制台日志输出，发现Spring Boot已经检测到class文件变化，重启了。

主要多了两个变化，LiveReload server用于监控Spring Boot应用文件变化，这是因为我们增加了“spring-boot-devtools”，另外启动时间变为0.5秒。为什么这么快启动？因为Spring Boot再次重启，避免了重启Tomcat Server，也避免了重启已经加载的Spring相关类，只重新加载变化的类。所以速度很快，基本上改完代码或者配置，就能立即进行调试。