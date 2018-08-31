---
title: 【译】Spring Boot @PropertySource 读取 YAML 文件
date: 2018-08-31 17:26:43
tags:
  - spring boot
---

Spring Boot 默认不支持`@PropertySource`读取yaml 文件，这也是Stackoverflow 上经常给予的标准答案。

![1535708009719](/images/1535708009719.png)

<!-- more -->

Spring 4.3 通过引入 [`PropertySourceFactory`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/support/PropertySourceFactory.html) 接口使之成为可能。`PropertySourceFactory` 是`PropertySource` 的工厂类。默认实现是 [`DefaultPropertySourceFactory`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/support/DefaultPropertySourceFactory.html)，可以构造`ResourcePropertySource` 实例。

可以通过普通的是实现构造 createPropertySource， 需要做两点:

- 导入resource 到Properties 对象。
- 构造 PropertySource 使用Properties。

具体例子：

```java
public class YamlPropertySourceFactory implements PropertySourceFactory {
    @Override
    public PropertySource<?> createPropertySource(@Nullable String name, EncodedResource resource) throws IOException {
        Properties propertiesFromYaml = loadYamlIntoProperties(resource);
        String sourceName = name != null ? name : resource.getResource().getFilename();
        return new PropertiesPropertySource(sourceName, propertiesFromYaml);
    }

    private Properties loadYamlIntoProperties(EncodedResource resource) throws FileNotFoundException {
        try {
            YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
            factory.setResources(resource.getResource());
            factory.afterPropertiesSet();
            return factory.getObject();
        } catch (IllegalStateException e) {
            // for ignoreResourceNotFound
            Throwable cause = e.getCause();
            if (cause instanceof FileNotFoundException)
                throw (FileNotFoundException) e.getCause();
            throw e;
        }
    }
}
```

**注意**：YAML 需要 SnakeYAML 1.18 或者更高版本。

@PropertySource 注解有一个 `factory` 属性，通过这个属性来注入 `PropertySourceFactory`，这里给出 `YamlPropertySourceFactory`的例子。

```java
@SpringBootApplication
@PropertySource(factory = YamlPropertySourceFactory.class, value = "classpath:blog.yaml")
public class YamlPropertysourceApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext ctx =
                SpringApplication.run(YamlPropertysourceApplication.class, args);

        ConfigurableEnvironment env = ctx.getEnvironment();
        env.getPropertySources()
                .forEach(ps -> System.out.println(ps.getName() + " : " + ps.getClass()));

        System.out.println("Value of `foo.bar` = " + env.getProperty("foo.bar"));
    }
}
```

**注意**：这里使用的是Spring Boot，但是对于Spring 4.3 及其以上版本同样适用。

翻译拙劣，欢迎指正。

reference:

[Use @PropertySource with YAML files](https://mdeinum.github.io/2018-07-04-PropertySource-with-yaml-files/)