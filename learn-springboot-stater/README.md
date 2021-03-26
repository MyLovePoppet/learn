# HelloSpring 
在maven内导入Spring支持如下：
```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.3.4.RELEASE</version>
    </dependency>
```
创建一个SpringBoot启动类：
```Java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
//使用该注解标注这是一个SpringBoot启动类
@SpringBootApplication
public class ServerApplication {
    public static void main(String[] args) {
        //调用SpringApplication的run方法
        SpringApplication.run(ServerApplication.class, args);
    }
}
```
接着在同各package下面定义一个Http接口
```Java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
//该注解表示Http接口
@RestController
public class ServerController {
    //显示的页面路径
    @RequestMapping("/HelloWorld")
    public String helloWorldIndex() {
        //返回到网页页面上输出的信息
        return "Hello, world!";
    }
}
```
如下图所示，一个最简单的web接口就创建好了，从浏览器内访问如下图所示：
![img.png](https://i.niupic.com/images/2020/11/30/934x.png)

# SpringBoot运行原理初探
>启动器 spring-boot-starter

在最开始导入的maven内包含一个名为```springboot-boot-starter-web```的包

其中springboot-boot-starter-xxx：就是spring-boot的场景启动器

spring-boot-starter-web：帮我们导入了web模块正常运行所依赖的组件；

SpringBoot将所有的功能场景都抽取出来，做成一个个的starter （启动器），只需要在项目中引入这些starter即可，所有相关的依赖都会导入进来 ， 我们要用什么功能就导入什么样的场景启动器即可 ；我们未来也可以自己自定义 starter；

>@SpringBootApplication

作用：标注在某个类上说明这个类就是SpringBoot的主配置类，SpringBoot就会运行该类的main方法来启动SpringBoot应用。

该注解内还能看到其他很多注解，如下图所示：

![img.png](https://i.niupic.com/images/2020/11/30/934H.png)

>@ComponentScan

这个注解在Spring中很重要,它对应XML配置中的元素。

作用：自动扫描并加载符合条件的组件或者bean，将这个bean定义加载到IOC容器中

>@EnableAutoConfiguration

@EnableAutoConfiguration：开启自动配置功能

以前我们需要自己配置的东西，而现在SpringBoot可以自动帮我们配置；@EnableAutoConfiguration告诉SpringBoot开启自动配置功能，这样自动配置才能生效；

在其中我们能找到导入配置文件的代码，如下：
```Java
protected List<String> getAutoConfigurations() {
    if (this.autoConfigurations == null) {
        this.autoConfigurations = SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class,
                this.beanClassLoader);
    }
    return this.autoConfigurations;
}
```
我们继续进行跟踪，发现导入的配置文件为：
```Java
  /**
    * The location to look for factories.
    * <p>Can be present in multiple JAR files.
    */
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
```
如下图所示：

![img.png](https://i.niupic.com/images/2020/12/01/93lm.png)

**结论：**

SpringBoot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值

将这些值作为自动配置类导入容器 ， 自动配置类就生效 ， 帮我们进行自动配置工作；

整个J2EE的整体解决方案和自动配置都在springboot-autoconfigure的jar包中；

它会给容器中导入非常多的自动配置类 （xxxAutoConfiguration）, 就是给容器中导入这个场景需要的所有组件 ， 并配置好这些组件 ；

有了自动配置类 ， 免去了我们手动编写配置注入功能组件等的工作；

# yaml配置注入

## yaml语法学习

SpringBoot使用一个全局的配置文件，配置文件名称是固定的

application.properties的语法结构为key=value

application.yml的语法结构为key：{空格}value

配置文件的作用 ：修改SpringBoot自动配置的默认值，因为SpringBoot在底层都给我们自动配置好了

比如我们可以在配置文件中修改Tomcat 默认启动的端口号！测试一下！

## 使用配置文件改变应用程序端口号

```yaml
server:
  port: 8081
```
将上述配置写入配置文件```application.yml```中，然后放入到maven的resources文件夹下，配置maven资源过滤以及编码，jdk编译版本如下：
```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
                <include>**/*.yml</include>
                <include>**/*.properties</include>
            </includes>
            <filtering>true</filtering>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.xml</include>
                <include>**/*.yml</include>
                <include>**/*.properties</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.0</version>
            <configuration>
                <source>11</source>
                <target>11</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```
如下图所示：

![img.png](https://i.niupic.com/images/2020/12/01/93mi.png)

后使用命令```mvn resources:resources```命令将配置文件过滤过去，重新启动Application，可以看到应用程序已经跑在了8081端口上：

![img.png](https://i.niupic.com/images/2020/12/01/93mk.png)

## yaml注入自定义配置
原来我们进行配置文件注入时使用的方法是使用@Value注解，如下：
```java
import lombok.Data;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Data
@Component
public class Dog {
    @Value("阿猫")
    private String name;
    @Value("10")
    private int age;
}
```

我们编写SpringBootTest测试如下：
```Java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest(classes = ServerApplication.class)
public class TestSpringBoot {
    @Autowired
    Dog dog;
    @Test
    public void contextLoads() {
        System.out.println(dog); //打印看下狗狗对象
    }
}
```
测试结果如下，可以看到数据已经成功注入进去了：

![img.png](https://i.niupic.com/images/2020/12/01/93na.png)

接着我们编写一个比较复杂一点的实体对象如下：
```java
@Data
@Component  //注册bean
@ConfigurationProperties(prefix = "person") //找到配置所在
public class Person {
    private String name;
    private int age;
    private boolean happy;
    private Date birth;
    private Map<String, Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```
我们在配置文件中编写配置如下：
```yaml
person:
  name: shuqy
  age: 20
  happy: true
  birth: 1999/12/10
  maps: {k1: v1,k2: v2}
  lists:
    - code
    - size
    - picture
  dog:
    name: 阿猫
    age: 1
```
测试如下，可以看到我们的Person对象也被成功注入了：

![img.png](https://i.niupic.com/images/2020/12/01/93pa.png)

## 加载指定的配置文件

我们可以使用@PropertySource注解进行自定义配置文件的注入，如```@PropertySource(value = "classpath:Person.properties")```，会进行找到Person.properties，并进行注入，将Person内需要注入的数据成员上加上@Value注解，如下：
```Java
@Component  //注册bean
@PropertySource(value = "classpath:Person.properties") //找到配置所在
public class Person {
    @Value("${name}")
    private String name;
}
```
最后我们的的数据也能成功注入，但是这种方式比较复杂，需要一个个指定对象，然后使用@Value进行注解，才能成功注入，如下为@ConfigurationProperties与@Value注解的对比：

|                      | @ConfigurationProperties | @Value           |
| :------------------- | :----------------------- | :--------------- |
| 功能                 | 批量注入配置文件中的属性 | 一个一个进行指定 |
| 松散绑定（松散语法） | 支持                     | 不支持           |
| SpEL                 | 不支持                   | 支持             |
| JSR303数据校验       | 支持                     | 不支持           |
| 复杂类型封装         | 支持                     | 不支持           |

**小结：**

1. @ConfigurationProperties只需要写一次即可，@Value则需要每个字段都添加
2. 松散绑定：这个什么意思呢? 比如我的yml中写的last-name，这个和lastName是一样的， - 后面跟着的字母默认是大写的。这就是松散绑定。可以测试一下
3. JSR303数据校验，这个就是我们可以在字段是增加一层过滤器验证，可以保证数据的合法性
4. 复杂类型封装，yml中可以封装对象，使用value就不支持

**结论：**

1. 配置yml和配置properties都可以获取到值 ， 强烈推荐 yml；

2. 如果我们在某个业务中，只需要获取配置文件中的某个值，可以使用一下 @value；

3. 如果说，我们专门编写了一个JavaBean来和配置文件进行一一映射，就直接使用@configurationProperties。

# JSR303数据校验及多环境切换
## JSR303数据校验
首先导入Maven环境
```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
        <version>2.3.4.RELEASE</version>
    </dependency>
```

SpringBoot中可以使用@Validated来进行数据校验，如果数据异常则会统一抛出异常，方便异常中心统一处理。如下例子表示我们的email属性只支持Email格式，其他格式将会直接抛出异常。

```Java
@Data
@Component  //注册bean
@ConfigurationProperties(prefix = "person")
@Validated//表示需要验证数据
public class Person {
    @Email(message = "邮箱格式错误")//错误信息
    private String email;
}
```
如果我们指定一个错误的邮箱格式的话，我们的程序就会抛出异常：

![img.png](https://i.niupic.com/images/2020/12/01/93q2.png)

如果指定的邮箱格式是正确的话，则会成功注入：

![img.png](https://i.niupic.com/images/2020/12/01/93q6.png)

如下为一些常见的验证注解：
```Java
@NotNull(message="名字不能为空")
private String userName;
@Max(value=120,message="年龄最大不能超过120")
private int age;
@Email(message="邮箱格式错误")
private String email;
 
 
//空检查
@Null       验证对象是否为null
@NotNull    验证对象是否不为null, 无法查检长度为0的字符串
@NotBlank   检查约束字符串是不是Null还有被Trim的长度是否大于0,只对字符串,且会去掉前后空格.
@NotEmpty   检查约束元素是否为NULL或者是EMPTY.
    
//Booelan检查
@AssertTrue     验证 Boolean 对象是否为 true  
@AssertFalse    验证 Boolean 对象是否为 false  
    
//长度检查
@Size(min=, max=) 验证对象（Array,Collection,Map,String）长度是否在给定的范围之内  
@Length(min=, max=) string is between min and max included.
 
 
//日期检查
@Past       验证 Date 和 Calendar 对象是否在当前时间之前  
@Future     验证 Date 和 Calendar 对象是否在当前时间之后  
@Pattern    验证 String 对象是否符合正则表达式的规则
 
 
.......等等
除此以外，我们还可以自定义一些数据校验规则
```
## 多环境切换
profile是Spring对不同环境提供不同配置功能的支持，可以通过激活不同的环境版本，实现快速切换环境，多环境切换一般有多配置文件和yaml多文档块两种实现方式。
### 多配置文件
我们在主配置文件编写的时候，文件名可以是 application-{profile}.properties/yml , 用来指定多个环境版本；

例如：

application-test.properties 代表测试环境配置

application-dev.properties 代表开发环境配置

但是Springboot并不会直接启动这些配置文件，它默认使用application.properties主配置文件；

我们需要通过一个配置来选择需要激活的环境：

```properties
#比如在配置文件中指定使用dev环境，我们可以通过设置不同的端口号进行测试；
#我们启动SpringBoot，就可以看到已经切换到dev下的配置了；
spring.profiles.active=dev
```
### yaml多文档块
和properties配置文件中一样，但是使用yml去实现不需要创建多个配置文件，更加方便了，如下为一个多文档块的yaml示例：
```yaml
#选择要激活那个环境块
spring:
  profiles:
    active: prod


---
spring:
  profiles: dev

person:
  name: shuqy-dev


---
spring:
  profiles: prod

person:
  name: shuqy-prod
```
上述我们激活的是prod文档块，我们测试注入属性数据结果如下，可以看到注入了正确的环境：

![img.png](https://i.niupic.com/images/2020/12/01/93qm.png)

## 配置文件加载位置
外部加载配置文件的方式十分多，我们选择最常用的即可，在开发的资源文件中进行配置！

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件：

优先级1：项目路径下的config文件夹配置文件

优先级2：项目路径下配置文件

优先级3：资源路径下的config文件夹配置文件

优先级4：资源路径下配置文件

**优先级由高到底，高优先级的配置会覆盖低优先级的配置；**

SpringBoot会从这四个位置全部加载主配置文件；互补配置。


# 自定义Starter

