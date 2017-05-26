# Spring注释 - @Autowired
@Autowired 是 Spring 注释中的一种，它可以应用于函数的变量，函数的构造函数，以及 set 方法上。并且它是通过类型来判定需要加载哪个 bean。
##变量
```
public class AnnotationTest {
	@Autowired
	private AnnotationObject annotationObject;
	
	//...
}
```
## 构造函数
```
public class AnnotationTest {
	private AnnotationObject annotationObject;
	
	@Autowired
	public AnnotationTest(AnnotationObject annotationObject) {
		this.annotationObject = annotationObject;
	}
	
	//...
}
```
## set 方法
```
public class AnnotationTest {
	private AnnotationObject annotationObject;
	
	@Autowired
	public void setAnnotationObject(AnnotationObject annotationObject) {
		this.annotationObject = annotationObject;
	}
	
	//...
}
```
## 异常情况
默认情况下，当没有对应的 bean 申明的时候，@Autowired 会失败，因为默认 @Autowired 是需要有依赖的，当然你也可以手动设置无依赖

```
public class AnnotationTest {
	private AnnotationObject annotationObject;
	
	@Autowired（required=false）
	public void setAnnotationObject(AnnotationObject annotationObject) {
		this.annotationObject = annotationObject;
	}
	
	//...
}
```
## XML Spring 设置
如果你仅仅是在代码中加入了注释，而没有做其他事情，这个注释是不会生效的。你需要使用 BeanPostProcessor 来配合注释，从而实现 Spring 的 IOC。其实在 Spring 中，注入（Inject）的方式有两种：

* 通过 xml 注入
* 通过 Annotaion 注入

但是需要注意的一点是：Annotation 注入是先于 xml 注入的，如果你两种方式都采用了的话，后者会将前者覆盖

如果你要使用 Annotation，那么你需要再 XML-based spring 的配置文件中，加入如下代码

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-2.5.xsd">
               
     <context:annotation-config/>
     
</beans>
```
其实以上的 post-processors 包括了：AutowiredAnnotationBeanPostProcessor, CommonAnnotationBeanPostProcessor, PersistenceAnnotationBeanPostProcessor, 以及 RequiredAnnotationBeanPostProcessor.

***参考资料***

[spring 3.0 doc](http://docs.spring.io/spring/docs/3.0.x/reference/beans.html#beans-annotation-config)

[spring-auto-wiring-beans-with-autowired](http://www.mkyong.com/spring/spring-auto-wiring-beans-with-autowired-annotation/)

[Spring 2.5 注释驱动的 IoC 功能](https://www.ibm.com/developerworks/cn/java/j-lo-spring25-ioc/)
