---
layout: post
title: 'Spring Boot 单元测试'
date: 2019-04-07
author: lipeng
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: SpringBoot
---

## spring-boot-starter-test

Spring Boot 提供了 spring-boot-starter-test 以供开发者使用单元测试，通过在 pom 中添加如下依赖引入：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```

spring-boot-starter-test 中提供了如下单元测试库：

- JUnit：一个 Java 语言的单元测试框架
- Spring Test & Spring Boot Test：为 Spring Boot 应用提供集成测试和工具支持
- AssertJ：支持流式断言的 Java 测试框架
- Hamcrest：一个匹配器库
- Mockito：一个 Java Mock 框架
- JSONassert：一个 JSON 断言库
- JsonPath：JSON XPath 库

![spring-boot-starter-test]({{site.baseUrl}}/assets/img/spring-boot-starter-test.png)

### 常用注解

- @RunWith(SpringRunner.class)

  该注解标签是 Junit 提供的，用来说明此测试类的运行者，这里用了 SpringRunner,它实际上继承了 SpringJUnit4ClassRunner 类，而 SpringJUnit4ClassRunner 这个类是一个针对 Junit 运行环境的自定义扩展，用来标准化在 Springboot 环境下 Junit4.x 的测试用例。

- @SpringBootTest

  该注解为 SpringApplication 创建上下文并支持 Spring Boot 特性，其 webEnvironment 提供如下配置：

  - Mock（默认）：加载 WebApplicationContext 并提供 Mock Servlet 环境，嵌入的 Servlet 容器不会被启动。
  - RANDOM_PORT：加载一个 EmbeddedWebApplicationContext 并提供一个真实的 servlet 环境。嵌入的 Servlet 容器将被启动并在一个随机端口上监听。
  - DEFINED_PORT：加载一个 EmbeddedWebApplicationContext 并提供一个真实的 servlet 环境。嵌入的 Servlet 容器将被启动并在一个默认的端口上监听（application.properties 配置端口或者默认端口 8080）。
  - NONE：使用 SpringApplication 加载一个 ApplicationContext，但是不提供任何的 servlet 环境。

- @MockBean

  在 ApplicationContext 里为一个 bean 定义一个 Mockito mock。

- @SpyBean

  定制化 Mock 某些方法。使用@SpyBean 除了被打过桩的函数，其它的函数都将真实返回。

- @WebMvcTest

  该注解被限制为一个单一的 controller，需要利用@MockBean 去 Mock 合作者（如 service）。

**Spring Boot 中单元测试类写在 src/test/java 目录下，可以通过 IDEA 自动创建测试类，快捷键为 Ctrl+Shift+T(Window)。**

## Controller 单元测试

### 使用模拟环境进行测试

默认情况下，@SpringBootTest 不会启动服务器，如果需针对此模拟环境测试 Web 端点，可以如下配置 MockMvc:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;
    @Test
    public void userMapping() throws Exception {
        String content = "{\"username\":\"pj_mike\",\"password\":\"123456\"}";
        mockMvc.perform(MockMvcRequestBuilders.request(HttpMethod.POST, "/user")
                        .contentType("application/json").content(content))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().string("ok"));
    }
}
```

MockMvc 实现了对 Http 请求的模拟，能够直接使用网络的形式，转换到 Controller 的调用，这样可以使得测试速度快、不依赖网络环境，而且提供了一套验证的工具，这样可以使得请求的验证统一而且很方便。

@AutoConfigureMockMvc: 该注解表示启动测试的时候自动注入 MockMvc, MockMvc 有以下几个基本的方法:

perform: 执行一个 RequestBuilder 请求，会自动执行 SpringMVC 的流程并映射到相应的控制器执行处理。
andExpect: 添加 RequsetMatcher 验证规则，验证控制器执行完成后结果是否正确。
andDo: 添加 ResultHandler 结果处理器，比如调试时打印结果到控制台。
andReturn: 返回相应的 MvcResult，进行自定义验证/进行下一步的异步处理。

#### 使用 MockMvcBuilder 构建 MockMvc 对象

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserControllerTest {
    @Autowired
    private WebApplicationContext web;
    private MockMvc mockMvc;

    @Before
    public void setupMockMvc() {
        mockMvc = MockMvcBuilders.webAppContextSetup(web).build();
    }
    @Test
    public void userMapping() throws Exception {
        String content = "{\"username\":\"pj_m\",\"password\":\"123456\"}";
        mockMvc.perform(request(HttpMethod.POST, "/user")
                        .contentType("application/json").content(content))
                .andExpect(status().isOk())
                .andExpect(content().string("ok"));
    }
}
```

### 使用真实 Web 环境进行测试

在@SpringBootTest 注解中设置属性 webEnvironment = WebEnvironment.RANDOM_PORT,每次运行的时候会随机选择一个可用端口。我们也可以还使用 @LoalServerPort 注解用于本地端口号。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class UserControllerTest {
    @Autowired
    private TestRestTemplate testRestTemplate;
    @Test
    public void userMapping() throws Exception {
        User user = new User();
        user.setUsername("pj_pj");
        user.setPassword("123456");
        ResponseEntity<String> responseEntity = testRestTemplate.postForEntity("/user", user, String.class);
        System.out.println("Result: "+responseEntity.getBody());
        System.out.println("状态码: "+responseEntity.getStatusCodeValue());
    }
}
```

## assertThat

JUnit 4.4 结合 Hamcrest 提供了一个全新的断言语法——assertThat。程序员可以只使用 assertThat 一个断言语句，结合 Hamcrest 提供的匹配符，就可以表达全部的测试思想。

assertThat 的基本语法如下：

`assertThat( [value], [matcher statement] );`

- value 是想要测试的变量值；
- matcher statement 是使用 Hamcrest 匹配符来表达的对前面变量所期望的值的声明，如果 value 值与 matcher statement 所表达的期望值相符，则测试成功，否则测试失败。

示例：

```java
字符相关匹配符
/**equalTo匹配符断言被测的testedValue等于expectedValue，
* equalTo可以断言数值之间，字符串之间和对象之间是否相等，相当于Object的equals方法
*/
assertThat(testedValue, equalTo(expectedValue));
/**equalToIgnoringCase匹配符断言被测的字符串testedString
*在忽略大小写的情况下等于expectedString
*/
assertThat(testedString, equalToIgnoringCase(expectedString));
/**equalToIgnoringWhiteSpace匹配符断言被测的字符串testedString
*在忽略头尾的任意个空格的情况下等于expectedString，
*注意：字符串中的空格不能被忽略
*/
assertThat(testedString, equalToIgnoringWhiteSpace(expectedString);
/**containsString匹配符断言被测的字符串testedString包含子字符串subString**/
assertThat(testedString, containsString(subString) );
/**endsWith匹配符断言被测的字符串testedString以子字符串suffix结尾*/
assertThat(testedString, endsWith(suffix));
/**startsWith匹配符断言被测的字符串testedString以子字符串prefix开始*/
assertThat(testedString, startsWith(prefix));
一般匹配符
/**nullValue()匹配符断言被测object的值为null*/
assertThat(object,nullValue());
/**notNullValue()匹配符断言被测object的值不为null*/
assertThat(object,notNullValue());
/**is匹配符断言被测的object等于后面给出匹配表达式*/
assertThat(testedString, is(equalTo(expectedValue)));
/**is匹配符简写应用之一，is(equalTo(x))的简写，断言testedValue等于expectedValue*/
assertThat(testedValue, is(expectedValue));
/**is匹配符简写应用之二，is(instanceOf(SomeClass.class))的简写，
*断言testedObject为Cheddar的实例
*/
assertThat(testedObject, is(Cheddar.class));
/**not匹配符和is匹配符正好相反，断言被测的object不等于后面给出的object*/
assertThat(testedString, not(expectedString));
/**allOf匹配符断言符合所有条件，相当于“与”（&&）*/
assertThat(testedNumber, allOf( greaterThan(8), lessThan(16) ) );
/**anyOf匹配符断言符合条件之一，相当于“或”（||）*/
assertThat(testedNumber, anyOf( greaterThan(16), lessThan(8) ) );
数值相关匹配符
/**closeTo匹配符断言被测的浮点型数testedDouble在20.0¡À0.5范围之内*/
assertThat(testedDouble, closeTo( 20.0, 0.5 ));
/**greaterThan匹配符断言被测的数值testedNumber大于16.0*/
assertThat(testedNumber, greaterThan(16.0));
/** lessThan匹配符断言被测的数值testedNumber小于16.0*/
assertThat(testedNumber, lessThan (16.0));
/** greaterThanOrEqualTo匹配符断言被测的数值testedNumber大于等于16.0*/
assertThat(testedNumber, greaterThanOrEqualTo (16.0));
/** lessThanOrEqualTo匹配符断言被测的testedNumber小于等于16.0*/
assertThat(testedNumber, lessThanOrEqualTo (16.0));
集合相关匹配符
/**hasEntry匹配符断言被测的Map对象mapObject含有一个键值为"key"对应元素值为"value"的Entry项*/
assertThat(mapObject, hasEntry("key", "value" ) );
/**hasItem匹配符表明被测的迭代对象iterableObject含有元素element项则测试通过*/
assertThat(iterableObject, hasItem (element));
/** hasKey匹配符断言被测的Map对象mapObject含有键值“key”*/
assertThat(mapObject, hasKey ("key"));
/** hasValue匹配符断言被测的Map对象mapObject含有元素值value*/
assertThat(mapObject, hasValue(value));
```

## 单元测试回滚

单元测试的时候，如果不想造成垃圾数据，可以开启事务功能，在方法或类头部添加 @Transactional 注解即可，在官方文档中对此也有说明:

> If your test is @Transactional, it rolls back the transaction at the end of each test method by default. However, as using this arrangement with either RANDOM_PORT or DEFINED_PORT implicitly provides a real servlet environment, the HTTP client and server run in separate threads and, thus, in separate transactions. Any transaction initiated on the server does not roll back in this case.

解读一下，在单元测试中使用 @Transactional 注解，默认情况下在测试方法的末尾会回滚事务。然而有一些特殊情况需要注意，当我们使用 RANDOM_PORT 或 DEFINED_PORT 这种安排隐式提供了一个真正的 Servlet 环境，所以 HTTP 客户端和服务器将在不同的线程中运行，从而分离事务，这种情况下，在服务器上启动的任何事务都不会回滚。

如果你想关闭回滚，只要加上 @Rollback(false)注解即可，@Rollback 表示事务执行完回滚，支持传入一个 value，默认 true 即回滚，false 不回滚。

#### 参考

SpringBoot 单元测试：<https://www.jianshu.com/p/813fd69aabee>

springboot 系列文章之使用单元测试：<https://juejin.im/post/5b95dbe46fb9a05cd7772503>

Spring Boot 干货系列：（十二）Spring Boot 使用单元测试：<http://tengj.top/2017/12/28/springboot12/#>
y