## Struts框架

### 定义：Struts是流行和成熟的基于MVC设计模式的Web应用程序框架。

Model1 = JSP + JavaBean

Model2 = JSP + Servlet + JavaBean

Struts中action就是Controller，Struts2是webwork的升级同时吸收两者的优势，不是一个全新的框架。

Struts2：servlet2.4 jsp2.0 java5(注解)

搭建struts2的步骤：

Jar -> 创建web项目 -> 完成配置  -> 创建action并测试

核心配置文件：web.xml struts.xml struts.properties

Struts2不再与Servlet API耦合，无需传入HttpServletRequest和HttpServletResponse

### struts2提供了3种方式访问Servlet API

1. ActionContext 
2. 实现Aware接口 
3. ServletActionContext

### 动态方法调用

1.指定method属性 2.感叹号方式 3.通配符方式 -> 解决action太多的问题

### 多个配置文件的方式 方法：include包含的方式

###吧默认Action - 解决无法匹配的Action

### 后缀名 struts.action.extension

### 接收参数 1.使用Action的属性接收 2.使用DomainModel接收 3.实现ModelDriven接口

代码地址:[struts](https://github.com/lemonjing/struts)