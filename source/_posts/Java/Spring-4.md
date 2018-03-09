---
title: Spring-4
date: 2017-01-01 09:00:04
tags: [Java, Spring]
---

整理自《Spring实战（第4版）》

**第 4 部分 Spring 集成**

<!-- more -->

# 第 15 章 使用远程服务

# 第 16 章 使用 Spring MVC 创建 REST API

# 第 17 章 Spring 消息

# 第 18 章 使用 WebSocket 和 STOMP 实现消息功能

# 第 19 章 使用 Spring 发送 Email

# 第 20 章 使用 JMX 管理 Spring Bean

JMX

> Java Management Extensions

使用 JMX 管理应用的核心组件是托管 bean （managed bean，MBean），即暴露特定方法的 JavaBean，这些方法定义了管理接口。

JMX 规范定义了 4 种类型的 MBean：

- 标准 MBean：标准 MBean 的管理接口是通过在固定的接口上执行反射确定的，bean 类会实现这个接口；
- 动态 MBean：动态 MBean 的管理接口是在运行时通过调用 `DynamicMBean` 接口的方法来确定的。因为管理接口不是通过静态接口定义的，因此可以在运行时改变；
- 开放 MBean：开放 MBean 是一种特殊的动态 MBean，其属性和方法只限定于原始类型、原始类型的包装类以及可以分解为原始类型或原始类型包装类的任意类型；
- 模型 MBean：模型 MBean 也是一种特殊的动态 MBean，用于充当管理接口与受管资源的中介。模型 Bean 并不像它们所声明的那样来编写。它们通常通过工厂生成，工厂会使用元信息来组装管理接口。

## 将 Spring bean 导出为 MBean

Spring 的 `MBeanExporter` 可以把 Spring bean 导出为 MBean 服务器内的模型 MBean。MBean 服务器（也称为 MBean 代理）是 MBean 生存的容器。对 MBean 的访问也是通过 MBean 服务器来实现的。

声明一个 `MBeanExporter`

```Java
@Bean
public MBeanExporter mbeanExporter(SpittleController spittleController) {
	MBeanExporter exporter = new MBeanExporter();
	Map<String, Object> beans = new HashMap<>();
	beans.put("spitter:name=SpittleController", spittleController);
	exporter.setBeans(beans);
	return exporter;
}
```

`MBeanExporter` 会假设它正在一个应用服务器中或提供 MBean 服务器的其他上下文中运行。若运行的是独立的应用或运行的容器没有提供 MBean 服务器，就需要在 Spring 上下文中配置一个 MBean 服务器。

在 XML 中使用 `<context:mbean-server>` 元素。或者声明一个 `MBeanServerFactoryBean` bean。可以将该 bean 装配到 `MBeanExporter` 的 `server` 属性中来指定 MBean 暴露到哪个 MBean 服务器中。

将 Spring bean 导出为 JMX MBean 之后，可以使用基于 JMX 的管理工具（如 JConsole 或 VisualVM）查看正在运行的应用，显示 bean 的属性并调用 bean 的方法。

为了对 MBean 的属性和操作获得更细粒度的控制，Spring 提供了几种选择：

- 通过名称来声明需要暴露或忽略的 bean 方法；
- 通过为 bean 增加接口来选择要暴露的方法；
- 通过注解标注 bean 来标识托管的属性和操作。

### 通过名称暴露方法

MBean 信息装配器（MBean info assembler）是限制哪些方法和属性将在 MBean 上暴露的关键。

其中一个 MBean 信息装配器是 `MethodNameBasedMBeanInfoAssembler`。另一个基于方法名称的装配器是 `MethodExclusionMBeanInfoAssembler`。它是前者的反操作，指定了不需要暴露为 MBean 托管操作的方法名称列表。

```java
@Bean
public MethodNameBasedMBeanInfoAssembler assembler1() {
	MethodNameBasedInfoAssembler assembler = new MethodNameBasedMBeanInfoAssembler();
	assembler.setManagedMethods(new String[] {
		"getSpittlesPerPage", "setSpittlesPerPage"
	});
	return assembler;
}

@Bean
public MethodExclusionMBeanInfoAssembler assembler2() {
	MethodExclusionMBeanInfoAssembler assembler = new MethodExclusionMBeanInfoAssembler();
	assembler.setIgnoreMethods(new String[] {
		"spittles"
	});
	return assembler;
}

/**
 * 将装配器装配进 MBeanExporter 中
 */
@Bean
public MBeanExporter mbeanExporter(SpittleController spittleController, MBeanInfoAssembler assembler) {
	MBeanExporter exporter = new MBeanExporter();
	Map<String, Object> beans = new HashMap<String, Object>();
	beans.put("spitter:name=SpittleController", spittleController);
	exporter.setBeans(beans);
	exporter.setAssembler(assembler);
	return exporter;
}
```

### 使用接口定义 MBean 的操作和属性

Spring 的 `InterfaceBasedMBeanInfoAssembler` 是另一种 MBean 信息装配器。通过使用接口选择 bean 的哪些方法需要暴露为 MBean 的托管操作。

假设有如下接口：

```java
public interface SpittleControllerManagedOperations {
	int getSpittlesPerPage();
	void setSpittlesPerPage(int spittlesPerPage);
}
```

只需用如下的 bean 替换掉之前基于方法名的装配器即可：

```java
@Bean
public InterfaceBasedMBeanInfoAssembler assembler() {
	InterfaceBasedMBeanInfoAssembler assembler = 
		new InterfaceBasedMBeanInfoAssembler();
	assembler.setManagedInterfaces(
		new Class<?>[] {SpittleControllerManagedOperations.class}
	);
	return assembler;
}
```

被托管的实现类不是必须实现 `SpittleControllerManagedOperation` 接口的。这个接口只是为了标识导出的内容。不过为了保证 MBean 和实现类之间保持一致，最好还是让实现类实现该接口。

### 使用注解驱动的 MBean

Spring 还提供另一种装配器 `MetadataMBeanInfoAssembler`，它可以使用注解标识哪些 bean 的方法需要暴露为 MBean 的托管操作和属性。

手动装配这种装配器很繁杂，简单的做法是使用 `<context:mbean-exporter>` 元素，用这种装配器代替之前的 `MBeanExporter`。

```xml
<context:mbean-exporter server="mbeanServer" />
```

```java
@Controller
//将 SpitteController 导出为 MBean
//objectName 标识了域（Spitter）和 MBean 的名称
@ManagedResource(objectName="spitter:name=SpittleController")
public class SpittleController {
	...
	//将 spittlesPerPage 暴露为托管属性
	@ManagedAttribute
	public void setSpittlesPerPage(int spittlesPerPage) {
		this.spittlesPerPage = spittlesPerPage;
	}

	@ManagedAttribute
	//不会把属性暴露为 MBean 的托管属性，因为此注解是严格限制方法的。
	//@ManagedOperation
	public int getSpittlePerPage() {
		return spittlePerPage;
	}
}
```

### 处理 MBean 冲突

Spring 提供了 3 种借助 `registrationBehaviorName` 属性来处理 MBean 名字冲突的机制：

- `FAIL_ON_EXISTING`：如果已存在相同名字的 MBean，则失败（默认）
- `IGNORE_EXISTING`：忽略冲突，不注册新的 MBean
- `REPLACING_EXISTING`：用新的覆盖旧的 MBean

```java
@Bean
pubilc MBeanExporter mbeanExporter(
	SpittleController spittleController,
	MBeanInfoAssembler assembler) {
	MBeanExporter exporter = new MBeanExporter();
	Map<String, Object> beans = new HashMap<>();
	beans.put("spitter:name=SpittleController", spittleController);
	exporter.setBeans(beans);
	exporter.setAssembler(assembler);
	exporter.setRegistrationPolicy(RegistrationPolicy.IGNORE_EXISTING);
	return exporter;
}
```

## 远程 MBean

为了满足以标准方式进行远程访问 JMX 的需求，JCP（Java Community Process）制订了 JSR-160:Java 管理扩展远程访问 API 规范（Java Management Extensions Remote API Specification）。该规范定义了 JMX 远程访问的标准，该标准至少需要绑定 RMI 和可选的 JMX 消息协议（JMX Messaging Protocol, JMXMP）。

### 暴露远程 MBean

```java
@Bean
public ConnectorServerFactoryBean connectorServerFactoryBean() {
	ConnectorServerFactoryBean csfb =
		new ConnectorServerFactoryBean();
	//（可选）绑定到一个 RMI 注册表
	csfb.setServiceUrl(
		"service:jmx:rmi://localhost/jndi/rmi://localhost:1099/spitter");
	return csfb;
}
```

`ConnectorServerFactoryBean` 会创建和启动 `JSR-160 JMXConnectorServer`。服务器默认使用 JMXMP 协议并监听 9875 端口，即绑定 `service:jmx:jmxmp://localhost:9875`。

### 访问远程 MBean

//TODO

### 代理 MBean

//TODO

### 处理通知

//TODO

### 监听通知

//TODO

# 第 21 章 借助 Spring Boot 简化 Spring 开发

## Spring Boot 简介

### 添加 Starter 依赖

|Starter|所提供的依赖|
|:--|:--|
|spring-boot-starter-actuator|spring-boot-starter、spring-boot-actuator、spring-core|
|spring-boot-starter-amqp|spring-boot-starter、spring-boot-rabbit、spring-core、spring-tx|
|spring-boot-starter-aop|spring-boot-starter、spring-aop、AspectJ Runtime、AspectJ Weaver、spring-core|
|spring-boot-starter-batch|spring-boot-starter、HSQLDB、spring-jdbc、spring-batch-core、spring-core|

//TODO

## 通过 Acutator 获取了解应用内部状况

```groovy
compile("org.springframework.boot:spring-boot-starter-actuator")
```

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-actuator</artifactId>
</dependency>
```

- `GET /autoconfig` 描述了 Spring Boot 在使用自动配置的时候所做出的决策；
- `GET /beans` 列出运行应用所配置的 bean；
- `GET /configprops` 列出应用中能够用来配置 bean 的所有属性及其当前的值；
- `GET /dump` 列出应用的线程，包括每个线程的栈跟踪信息；
- `GET /env` 列出应用上下文中所有可用的环境和系统属性变量；
- `GET /env/{name}` 展现某个特定环境变量和属性变量的值；
- `GET /health` 展现当前应用的健康状况；
- `GET /info` 展现应用特定的信息；
- `GET /metrics` 列出应用相关的指标，包括请求特定端点的运行次数；
- `GET /metrics/{name}` 展现应用特定指标项的指标状况；
- `POST /shutdown` 强制关闭应用；
- `GET /trace` 列出应用最近请求相关的元数据，包括请求和相应头。