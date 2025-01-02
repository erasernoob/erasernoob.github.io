---
title: '初识springMVC三层架构'
date: 2024-09-16T18:57:29+08:00
draft: false
layout: "post"
images: 
  - "/avatar.jpg"   # 设置封面图
description: ""     
categories: ["技术原理"]
tags: [""]
---

# SpringMVC

## MVC理论基础

Web的三层架构：

- 表示层：任何页面的返回和数据填充都是靠着表示层来完成的。`post`&`get`
- 业务逻辑层：业务逻辑层主要用于数据处理，表示层向逻辑层索要数据，之后将数据添加到前端页面的上下文中。
- 数据访问层：底层的数据库的连接和数据的提取。

MVC详细解释：

- M:(Model)指的是业务模型，也就是我们在编写程序时为了封装数据传递所建立的**实体类**。
- V: (View)指的是用户界面，前端界面。
- C:(Controller)控制器对应的是**Servlet**的基本功能，处理请求，返回响应。`doPost` `doGet`

MVC将三者之间进行解耦，最后将整个页面进行渲染一起打包给网页页面。

## @RequestMapping

`path`属性，决定了当前方法处理的请求路径，并且路径必须要全局唯一。同时这个属性也可以是一个数组，即数组当中的所有路径都可以通过该方法进行处理。 

**将路径前缀注解写在类的名上，则当前类下的所有方法处理的请求路径都要加上类的前缀**。

```java
@Controller
@RequestMapping("/yyds")
public class MainController {
    @RequestMapping({"/index", "/test"})
    public ModelAndView index(){
        return new ModelAndView("index");
    }
}
```

 路径也可以通过通配符进行匹配：

- "?"：表示任意一个字符
- "*"：表示任意0-n个字符
- "**"：表示当前当前目录或者基于当前目录的多级目录

`method`属性，请求方法的类型，可以限定请求的方法，只能通过特定的方式来发送请求。效果和响应的`PostMapping` `GetMapping`相同。

`params`属性，限定参数，必须要携带相对应的参数，否则无法访问。

`header`属性用法与`params`一致，但是它要求的是请求头中需要携带什么内容，比如：

```java
@RequestMapping(value = "/index", headers = "!Connection")
public ModelAndView index(){
    return new ModelAndView("index");
}
```

所以如果在这里携带了Connection属性，就无法访问了。 

## @RequestParam

该注解则可以获取到请求中的参数。

## 重定向和请求转发

只需要加前缀

## Bean的web作用域

和Spring中的Bean的作用域不同，不只有两种，在SpringMVC中的作用域继续被细分了。

- request：将类作用域修改为`@RequestScope`后，每一次不同的请求结束之后，相应的Bean也会消失，也就是说下一次得到的Bean创建出来的实例将会不同。
- session：将类作用域修改为`@SessionScope`后，其每一次会话，结束之后Bean就会销毁消失。（会话指打开浏览器到关闭浏览器）

## Interceptor拦截器

拦截器和JavaWeb中的filter类似，但是：

- 他的位置并不是处在Servlet之前，而是请求在通过Servlet之后所到达的控制器之前上。
- 他在DispathcherServlet将请求交给对应的Controller中的方法之前进行拦截处理。
- 不会拦截静态资源，只会拦截所有Controller中定义好了的请求映射对应的请求。

### 创建拦截器

​    创建拦截器需要实现`HndlerInterceptor`接口,实现方法：

- `boolean preHandler`开始处理之前，`只有返回true才会继续，否则直接结束`
- `postHandle`在处理完一个控制器执行完毕之后，开始返回。
- `afterCompletion` 在`DispacherServlet`完全处理完成请求之后被调用

```java
// 生命周期
InterceptorA - preHandle
InterceptorB - preHandle
[Controller Method Executed]
InterceptorB - postHandle
InterceptorA - postHandle
InterceptorB - afterCompletion
InterceptorA - afterCompletion
```



创建完成之后，在主配置类进行添加拦截器，设定需要拦截的页面请求。 

### 多级拦截器 

多个拦截器执行的顺序，和栈类似，一号先进则最后再出。也可以通过`addInterceptor` `order`方法设定拦截器的优先级。

## 异常处理

在处理请求映射器的抛出的异常时，我们应该自定义一个异常处理控制器，一旦出现了指定的异常，就会跳到该异常处理控制器。

## JSON数据格式与Axios请求

