# SpringBoot网页开发
## SpringBoot-web资源导入分析
```java
//WebMvcAutoConfiguration.java  ---320行
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/")
                .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
                .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
                .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
}
```
可以看到其首先注册了一个类似于```/webjars/**```的内容，那么逐步来进行分析
### 第一部分 /webjars/**
在webjars的官网上https://www.webjars.org/，我们可以使用类似Maven的形式导入一些静态资源，如导入jquery:
```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.6.0</version>
</dependency>
```
我们在pom.xml内导入jquery的pom，查看对应的目录结构：

![img.png](https://i.loli.net/2021/03/26/Ij4GStwKc1lNJDx.png)

可以看到jquery的路径类似于"jquery-3.6.0.jar!\META-INF\resources\webjars\jquery\3.6.0\jquery.js"，与上述代码中的```addResourceLocations("classpath:/META-INF/resources/webjars/")```相对应。在webjars官网内所有这种静态资源都遵循```classpath:/META-INF/resources/webjars/```这种地址。后续只需要使用路径"/webjars/***"即可访问对应的静态资源，如下图所示

![img.png](https://i.loli.net/2021/03/26/pOkeWM4UyAlP5oT.png)

### 第二部分 MVCProperties与ResourceProperties
- 在MVCProperties内的```staticPathPattern```字段，其值为"/**"
- ResourceProperties类内能看到还具有存放静态资源的路径：
    ```java
    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
            "classpath:/resources/", "classpath:/static/", "classpath:/public/" };
    ```
### 总结
SpringBoot-web有如下方式处理静态资源：
- /webjars/**                   --> localhost:8080/webjars/**
- /public, /resource, /static   --> localhostt:8080/

优先级为：resource-->static（默认）-->public 

测试：在resource文件夹下新建一个hello.js文件，内容为Hello, world!，测试使用地址访问localhost:8080/hello.js，结果如下：

![img.png](https://i.loli.net/2021/03/26/m7RLsfHCrBNuiW8.png)

## 首页定制
源码：
```java
//WebMvcAutoConfiguration.java  432行
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
        FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
    WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
            new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
            this.mvcProperties.getStaticPathPattern());
    welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
    welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
    return welcomePageHandlerMapping;
}

private Optional<Resource> getWelcomePage() {
    String[] locations = getResourceLocations(this.resourceProperties.getStaticLocations());
    return Arrays.stream(locations).map(this::getIndexHtml).filter(this::isReadable).findFirst();
}

private Resource getIndexHtml(String location) {
    return this.resourceLoader.getResource(location + "index.html");
}

private boolean isReadable(Resource resource) {
    try {
        return resource.exists() && (resource.getURL() != null);
    }
    catch (Exception ex) {
        return false;
    }
}
```
可以看到如果不进行配置时，```getWelcomePage()```函数会对上述静态资源地址内找到第一个index.html文件，然后返回。

我们在static文件夹下建立一个index.html文件，然后直接访问localhost:8080，会直接返回我们这个index.html文件，如下图所示：

![img.png](https://i.loli.net/2021/03/26/TYMhcvOGLK9uCtW.png)

# 模板引擎
模板引擎的概念类似于以前的JSP，其工作过程如下:

![img.png](https://i.loli.net/2021/03/26/Wj2mObekoraXyzv.png)

源码
```java
//ThymeleafProperties.java 
private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

public static final String DEFAULT_PREFIX = "classpath:/templates/";

public static final String DEFAULT_SUFFIX = ".html";
```
上述源码表示其默认的目录为templates，默认的文件结尾为.html。

现在将index.html放到templates目录下，为其建立一个Controller，如下：
```java
@Controller
public class IndexController {
    @RequestMapping("/index")
    public String index() {
        return "index";
    }
}
```
测试如下：

![image.png](https://i.loli.net/2021/03/26/4wkxib3YzJsqPBR.png)

一些简单的使用方式：
Variable Expressions: ${...}
Selection Variable Expressions: *{...}
Message Expressions: #{...}
Link URL Expressions: @{...}
Fragment Expressions: ~{...}

测试模板引擎如下：
页面数据：
```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
        "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Title</title>
</head>
<body>
<h1>This is title page!</h1>
<!--可以接管所有的html元素-->
<div th:text="${msg}"></div>
</body>
</html>
```
Controller:
```java
@Controller
public class IndexController {
    @RequestMapping("/index")
    public String index(Model model) {
        //替换
        model.addAttribute("msg", "hello, springboot");
        return "index";
    }
}
```
运行结果如下：

![image.png](https://i.loli.net/2021/03/26/3aoNEGLW9mPYyUT.png)

测试转义和each操作符

![image.png](https://i.loli.net/2021/03/26/nzc3PXiN6jeoZwp.png)

# 拓展SpringMVC
SpringBoot官方说法：

Spring Boot provides auto-configuration for Spring MVC that works well with most applications.
If you want to keep Spring Boot MVC features and you want to add additional MVC configuration (interceptors, formatters, view controllers, and other features), you can add your own @Configuration class of type WebMvcConfigurer but without @EnableWebMvc.
If you want to take complete control of Spring MVC, you can add your own @Configuration annotated with @EnableWebMvc.

只需建立一个自己的MVCConfigure类用于继承WebMvcConfigurer类，然后使用@Configuration注解，不使用@EnableWebMvc

如下为我们添加的一个视图控制器，将/indextest页面跳转到/index
```java
public class MyMVCConfig implements WebMvcConfigurer {
    //添加一个视图控制器
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //将/indextest页面跳转到/index
        registry.addRedirectViewController("/indextest", "/index");
    }

}
```
测试效果如下：

![image.png](https://i.loli.net/2021/03/26/h4nVxvCwzSpuqHe.png)

## 为什么说拓展SpringMVC不能加@EnableWebMvc注解
因为```WebMvcAutoConfiguration```类上有```@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)```注解，表示没有导入```WebMvcConfigurationSupport```类这个自动配置类才会生效，但是```@EnableWebMvc```注解内有这样一句话：```@Import(DelegatingWebMvcConfiguration.class)```，表示导入```DelegatingWebMvcConfiguration```类，这个```DelegatingWebMvcConfiguration```类是继承自```WebMvcConfigurationSupport```，这样一来```WebMvcAutoConfiguration```自动配置类就会自动失效了。