# Spring基础

## Spring Bean的声明周期

1.  通过反射调用Bean的构造方法来实例化Bean
2.  通过反射来注入Bean的属性值
3.  进行Aware注入，让Bean能够感知自己所处的环境，利用相关的容器资源
    -   如果Bean实现了BeanNameAware接口，则会调用Bean的setBeanName方法
    -   如果Bean实现了BeanFactoryAware接口，则会调用Bean的setBeanFactory方法
    -   如果Bean实现了ApplicationContext接口，则会调用Bean的setApplicationContext方法
4. 

![SpringBean生命周期](img/SpringBean生命周期.png)

## Srping Bean的作用域

Spring Bean的作用域

- singleton（默认）：以单例模式创建，每次从IOC容器获取的Bean实例都是同一个
- prototype：每次从IOC容器获取Bean实例时都会创建一个新的Bean实例
- request：为每个HTTP请求都创建一个新的Bean实例
- session：为每个Session都创建一个新的Bean实例
- global-session：所有Session共享同一个Bean

## SpringMVC的工作流程

第一步：发起请求到前端控制器(DispatcherServlet)
第二步：前端控制器请求HandlerMapping（处理器映射器）查找 Handler可以根据xml配置、注解进行查找
第三步：处理器映射器HandlerMapping向前端控制器返回Handler
第四步：前端控制器调用处理器适配器去执行Handler
第五步：处理器适配器去执行Handler
第六步：Handler执行完成给适配器返回ModelAndView
第七步：处理器适配器向前端控制器返回ModelAndView，ModelAndView是springmvc框架的一个底层对象，包括 Model和view
第八步：前端控制器请求视图解析器去进行视图解析，根据逻辑视图名解析成真正的视图(jsp)
第九步：视图解析器向前端控制器返回View
第十步：前端控制器进行视图渲染视图渲染将模型数据(在ModelAndView对象中)填充到request域
第十一步：前端控制器向用户响应结果