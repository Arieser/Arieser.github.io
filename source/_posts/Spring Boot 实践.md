---
title: Spring Boot 实践
date: 2018-08-20 15:08:00
tags:
  - spring boot
---

使用Spring  Boot 已经好几年了，看了一篇总述文章，觉得很不错，简要记录一下，并加入了自己使用过程的一些敬仰。

![1534944048141](/images/1534944048141.png)

<!-- more -->

1. ### 使用自定义BOM来维护第三方依赖

   使用Spring IO Platform 维护项目，在此基础上编写自己的基础项目platform-bom

   ```xml
   <dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>io.spring.platform</groupId>
              <artifactId>platform-bom</artifactId>
              <version>Cairo-SR3</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
   </dependencyManagement>
   ```

   [Spring IO Platform](https://www.cnblogs.com/chenpi/p/6295855.html) 介绍：Spring IO Platform，简单的可以认为是一个依赖维护平台，该平台将相关依赖汇聚到一起，针对每个依赖，都提供了一个版本号；这些版本对应的依赖都是经过测试的，可以保证一起正常使用。

2. ### 使用自动配置

   主要是依赖spring boot starters来集成各种组件，如redis，mongodb等

3. ### 使用Spring Boot 自动配置

4. **正确设计代码目录结构**

   - 避免使用默认包。确保所有内容（包括你的入口点）都位于一个名称很好的包中，这样就可以避免与装配和组件扫描相关的意外情况；
   - 将Application.java（应用的入口类）保留在顶级源代码目录中；
   - 我建议将控制器和服务放在以功能为导向的模块中，但这是可选的。一些非常好的开发人员建议将所有控制器放在一起。不论怎样，坚持一种风格！

5. **保持`@Controller`的简洁和专注**

   设计参照 [GRASP](https://en.wikipedia.org/wiki/GRASP_(object-oriented_design)#Controller)

6. **熟悉并发模型**

   在Spring Boot中，Controller和Service是默认是单例。如果你不小心，这会引入可能的并发问题。 你通常也在处理有限的线程池。请熟悉这些概念

7. **加强配置管理**

   使用spring cloud config，apollo， config server 等管理配置

8. **提供全局异常处理**

   两种方案：

   - 使用HandlerExceptionResolver定义全局异常处理策略
   - 添加@ExceptionHandler注解

9. **使用日志框架**

   配置和slf4j类似，默认使用logback-spring.xml文件，略

10. **测试代码**

  主要测试模块：

  - **spring-boot-test**：支持测试的核心内容
  - **spring-boot-test-autoconfigure**：支持测试的自动化配置

  调用示例：

  ```java
  @RunWith (SpringRunner.class)
  @SpringBootTest
  public class Test{
      @Autowired
      private TestRestTemplate testRestTemplate;
  
      @Test
      public void testDemo () {｝
  }
  ```

  文件上传测试

  ```java
  @Test
  public void  upload() throws Exception {
      Resource resource =  new FileSystemResource("");
      MultiValueMap multiValueMap = new LinkedMultiValueMap();
      multiValueMap.add("username", "java");
      HttpHeaders postHead = new HttpHeaders();
      String body = testRestTemplate.postForEntity("/abc", new HttpEntity<>(multiValueMap, postHead), String.class).getBody();
  }
  ```

  文件下载测试

  ```java
  @Test
  public void upload() throws Exception {
      Resource resource = new FileSystemResource("");
      MultiValueMap multiValueMap = new LinkedMultiValueMap();
      multiValueMap.add("username", "java");
      HttpHeaders postHead = new HttpHeaders();
      ResponseEntity<byte[]> response = testRestTemplate
          .exchange("/test/download?username={1}", HttpMethod.GET, new HttpEntity<>(postHead), byte[].class, new String[]{"admin"});
      if (response.getStatusCode() == HttpStatus.OK) {
          Files.write(response.getBody(), new File("/home/abc.txt"));
      }
  }
  ```

11. **使用测试切片**（未使用过）

    参考 https://spring.io/blog/2016/08/30/custom-test-slice-with-spring-boot-1-4

12. **Runner 启动器**

    使用`CommandLineRunner`可以在Spring Boot 启动时运行特定代码

13. **资源文件过滤问题**

    使用继承Spring Boot时，如果要使用Maven resource filter过滤资源文件时，资源文件里面的占位符为了使${}和Spring Boot区别开来，此时要用@...@包起来，不然无效。另外，@...@占位符在yaml文件编辑器中编译报错，所以使用继承方式有诸多问题

reference:

[两年摸爬滚打 Spring Boot，总结了这 16 条最佳实践](https://my.oschina.net/u/3906750/blog/1930397)