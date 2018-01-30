---
layout:

title: 6-EnableAutoConfiguration深入分析

date: 2017-02-06

updated: 2017-02-06

tags:
- SpringBoot
- SpringBoot与微服务
- SpringBoot实战与原理
- Spring
- 原理

categories: SpringBoot实战与原理分析

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---


# 6-EnableAutoConfiguration深入分析

[TOC]

- @SpringBootApplication由以下注解组成

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
)}
)
```

## 6.1 加载其他项目下的bean

- 在classpath下创建目录META-INF/spring.factories
- 在META-INF/spring.factories指定项目外的配置类或其他模块的配置类org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.clsaa.core.bean.RunnableConfiguration
- 等于号后面可以加多个配置类,用逗号隔开
- 可在@EnableAutoConfiguration(exclude=XXX.class)进行排除

```
	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
			AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
		Assert.notEmpty(configurations,
				"No auto configuration classes found in META-INF/spring.factories. If you "
						+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
```

- 实际上EnableAutoConfiguration使用了EnableAutoConfigurationImportSelector,前面讲到这个类返回一个字符串数组,把类名以字符串的形式返回,只要返回的类名就被导入到spring容器中管理起来