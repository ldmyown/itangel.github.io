---
layout: post
title: beetl与springboot结合实现
category: beetl
tags: [beetl]
---



Springboot目前功能非常强大，此处记录一下springboot集成使用beetl模板的方法。



## 项目搭建

### 创建一个简单的springboot项目

在pom文件中引入邮件 **beetl-framework-starter**的依赖

```xml
<dependency>
	<groupId>com.ibeetl</groupId>
	<artifactId>beetl-framework-starter</artifactId>
	<version>1.1.81.RELEASE</version>
</dependency>
```

### 在templates中创建模板文件index.btl

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="description" content="">
    <meta name="viewport" content="width=device-width">
</head>

<body>
hello ${beetl}
</body>


~~~



### 创建controller

~~~java
@Controller
public class IndexController {

    @GetMapping("/")
    public String index(HttpServletRequest request){
        request.setAttribute("beetl", "telangel");
        return "index.btl";
    }
}
~~~



至此，beetl与springboot的简单结合就完成了，在浏览器中访问，就可以通过模板渲染相应的数据。



> beetl-framework-starter 模板的根目录默认为springboot的templates目录，默认处理以btl结尾的视图。
>
> 如果需要修改，可以在springboot的配置文件application.properties配置。



### 配置beetl的参数

~~~properties
# 禁用beetlsql  默认为true 集成beetlsql
beetlsql.enabled=false
# 禁用beetl  默认为true，集成beetl模板
beetl.enabled=false
# 自动检查模板变化， 默认为true
beetl-beetlsql.dev = true
# 默认为btl，表示只处理视图后缀为btl的模板，这里如果不配置，则只能识别btl结尾的模板文件
beetl.suffix = btl
~~~



### 定制beetl

- 实现BeetlTemplateCustomize来定制Beetl

  ~~~java
  @Configuration
  public class MyBeetlConfig {

      @Bean
      public BeetlTemplateCustomize beetlTemplateCustomize(){
          return (groupTemplate) -> {
              Map<String, Object> map = new HashMap<>();
              map.put("user", "lid");
              groupTemplate.setSharedVars(map);
          };
      }
  }
  ~~~

- 配置自己的模板引擎

  通常情况下，beetl starter的配置已经足够使用了，如果需要自己配置模板引擎，需要配置

  BeetlGroupUtilConfiguration，和 BeetlSpringViewResolver，配置代码参考

  ~~~java
   @Bean(name = "beetlConfig")
   public BeetlGroupUtilConfiguration getBeetlGroupUtilConfiguration() {
       BeetlGroupUtilConfiguration beetlGroupUtilConfiguration = new BeetlGroupUtilConfiguration();
       //获取Spring Boot 的ClassLoader
       ClassLoader loader = Thread.currentThread().getContextClassLoader();
       if(loader==null){
              loader = BeetlConf.class.getClassLoader();
       }
       beetlGroupUtilConfiguration.setConfigProperties(extProperties);//额外的配置，可以覆盖默认配置，一般不需要
       ClasspathResourceLoader cploder = new ClasspathResourceLoader(loader,
                  templatesPath);
       beetlGroupUtilConfiguration.setResourceLoader(cploder);
       beetlGroupUtilConfiguration.init();
       //如果使用了优化编译器，涉及到字节码操作，需要添加ClassLoader
       beetlGroupUtilConfiguration.getGroupTemplate().setClassLoader(loader);
          return beetlGroupUtilConfiguration;
       }

          @Bean(name = "beetlViewResolver")
          public BeetlSpringViewResolver getBeetlSpringViewResolver(@Qualifier("beetlConfig") BeetlGroupUtilConfiguration beetlGroupUtilConfiguration) {
                  BeetlSpringViewResolver beetlSpringViewResolver = new BeetlSpringViewResolver();
                  beetlSpringViewResolver.setContentType("text/html;charset=UTF-8");
                  beetlSpringViewResolver.setOrder(0);
                  beetlSpringViewResolver.setConfig(beetlGroupUtilConfiguration);
                  return beetlSpringViewResolver;
          }
  ~~~

  ​

  ​