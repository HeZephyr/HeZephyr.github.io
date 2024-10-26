# 配置和排查 Lombok 在 IDEA 中使用的详细步骤

在日常开发中，Java 代码常常需要大量的样板代码，比如 `getter`、`setter`、`toString` 等方法。Lombok 是一个 Java 库，可以通过注解的方式，自动生成这些常见的代码，从而让代码更加简洁、清晰。比如，我们可以通过 `@Getter` 和 `@Setter` 注解自动生成 `getter` 和 `setter` 方法，通过 `@ToString` 自动生成 `toString` 方法。Lombok 的常用注解包括：

- `@Getter`：生成 `getter` 方法。
- `@Setter`：生成 `setter` 方法。
- `@ToString`：生成 `toString` 方法。
- `@Data`：生成 `getter`、`setter`、`toString`、`equals` 和 `hashCode` 方法。
- `@Builder`：提供构建者模式（Builder pattern）。

下面是配置和排查 Lombok 在 IDEA 中使用的详细步骤。

1. 安装 Lombok 插件

	![image-20241026105132473](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/image-20241026105132473.png)

2. 开启 Annotation Processing

	![image-20241026105218772](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/image-20241026105218772.png)

3. 在 pom.xml 中添加 Lombok 依赖

	在 Maven 项目中，需要在 `pom.xml` 中添加 Lombok 的依赖：

	```yml
	&lt;dependencies&gt;
	    &lt;!-- Lombok dependency --&gt;
	    &lt;dependency&gt;
	        &lt;groupId&gt;org.projectlombok&lt;/groupId&gt;
	        &lt;artifactId&gt;lombok&lt;/artifactId&gt;
	        &lt;version&gt;1.18.28&lt;/version&gt; &lt;!-- 根据需要选择版本 --&gt;
	        &lt;scope&gt;provided&lt;/scope&gt;
	    &lt;/dependency&gt;
	&lt;/dependencies&gt;
	
	```

	添加依赖后，IDEA 将会自动下载 Lombok 库。如果依赖没有自动下载，可以执行下一步来手动刷新项目。

4. 刷新 Maven 项目

	在 IDEA 中，点击右侧的 Maven 面板，找到你的项目，点击 `Reload All Maven Projects`（通常是一个循环箭头图标）。这将强制 Maven 刷新并重新下载所有依赖。完成后，IDEA 就可以正常识别 Lombok 的注解了。

	![image-20241026105404970](https://raw.githubusercontent.com/HeZephyr/NewPicGoLibrary/main/img/image-20241026105404970.png)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/06.%E9%85%8D%E7%BD%AE%E5%92%8C%E6%8E%92%E6%9F%A5lombok%E5%9C%A8idea%E4%B8%AD%E4%BD%BF%E7%94%A8%E7%9A%84%E8%AF%A6%E7%BB%86%E6%AD%A5%E9%AA%A4/  

