# spring boot 学习概览
1. 自动配置

2. 起步依赖之类


## 核心注解类
1. @SpringBootApplication 用在启动类里面
2. @Configuration 通过bean对象的操作替代spring当中的xml
3. @SpringBootConfiguration 通过bean对象来获取配置信息
4. @EnableAutoConfiguration 完成初始化环境的配置，比如spring mvc -- starter之类文件的配置
5. @conponentscan spring 组件扫描 ，替代spring组件扫描


controller层返回的东西都是 /方法映射到上面 

跟视图一起合作




# controller 
我的疑问点就是在什么时间节点会触发这个请求参数
就是比如requestMapping 之类的东西， 什么时候回触发
>我觉得就是用户在输入框当中输入参数，获取网页的时候就会触发这个问题，可能性,


controller层

get 之类方法的参数是由用户输如的， 

前端完成的工作就是 将用户输入的东西映射成为一个对象，然后controller层里面再调用service层面的东西，然后service 再调用mapper层面的东西
 
 
 return list 返回一个查询列表

# domian层面的东西
一般来讲，domain层面的东西，我们都是使用对象我们


get和post 请求的返回列表还不一样

get请求是返回一个结果，所有我们一般都是，一般都是列表类的情况，而post 都是封装成为一个返回的东西，确保是否

@RequestParam("id") 表明这个参数需要用户输入


## mybatis层开发

mybatis整合spring boot 

先比那些mapper接口和xml文件以及

mapper层代替dao, 层

```
// mapper 接口类扫描包配置
@MapperScan("org.spring.springboot.dao")
```

@SpringBootApplication
//mapper 接口类扫描包配置
@MapperScan("org.spring.springboot.dao")

# test测试类改怎么写


# dubbo
1. 
