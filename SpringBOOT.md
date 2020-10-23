#                                                            SpringBOOT

## 一、创建工程所需步骤

### 1.maven工程创建

一般来说，需要导入大量依赖包的工程，都交由maven来管理，所以，这里首先创建maven工程，通过导入相关的Spring BOOT依赖包，实现创建SpringBOOT工程。再导入相关依赖之后，通过main方法作为主程序入口启动Sping BOOT工程

```java
//@SpringBootApplication--标识SoringBOOT工程主程序注解
//主程序扫描所在包及以下子包，放入Sring容器中
@SpringBootApplication
public class MallApplication {

    public static void main(String[] args) {
        SpringApplication.run(MallApplication.class, args);
    }

}
```

```xml
  <!--SpringBOOT核心依赖，父项目管理其他依赖的默认版本号-->
   <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
    </parent>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
```

### 2.开发工具提供的快捷创建方式——Spring initializr

### 3.常用注解

```java
@SpringBootApplication  //主程序入口
@MapperScan(basePackages = {"mall.**.dao"})  //扫描Mapper Bean
@ComponentScan(basePackages = "mall.")  //扫描所有Bean
@RestController  //标识Controller层，并将此类所有返回内容返回给客户端/浏览器，等同于@Controller + @responsbody 
@responsbody  //将数据返回给客户端。浏览器时标识注解
@RequestParam  //获取请求参数，当方法入参与请求参数不一致时，可通过这种注解获取到请求数据封装给入参
@PathVariable  //获取Path类型请求数据
@RequestBody  //将请求的json数实例化
@Component  //通用注解，一般用于普通的pojo类实例化到容器中（配置文件）
```



## 二、Spring Boot配置文件

通常，开发工具创建的Spring Boot工程，配置文件格式默认为properties；不过我们可以使用yml格式文件进行配置，这种文件格式注重数据存储，更简洁。

```java
//自定义测试实体类

@Component
@ConfigurationProperties(prefix = "test") //指定注入属性值
public class test {

    int age ;
    String name ;
    Date birthday;

    @Override
    public String toString() {
        return "test{" +
                "age=" + age +
                ", name='" + name + '\'' +
                ", birthday=" + birthday +
                '}';
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
}

```

```yml
# 作为数据存储的文件格式，就可以在配置指定类的参数值
test:
    age: 12
    name: ikun
    birthday: 2020/12/12
```

```java
//测试类
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(classes = MallApplication.class)
class MallApplicationTests {

    @Autowired
    test test;

    @Test
    void contextLoads() {
        System.out.println(test);
    }
}
```

## 三、基本工作原理

相对于SSM，Spring Boot在POM中引入了父依赖，在父依赖中定义了大量第三方jar包，因此在导入受Spring Boot管理的依赖时，不需要关注版本，只有不在管理范围内的依赖需要写入版本号。

SpringBoot运行项目时，只需要启动注解@SpringBootApplication的主程序，其中定义了**@SpringBootConfiguration这个注解只是更好区分这是SpringBoot的配置注解，本质还是用了Spring提供的@Configuration注解**以及**@AutoConfigurationPackage**——自动加载每个场景所需要的组件，其本质还是Spring工作模式。

