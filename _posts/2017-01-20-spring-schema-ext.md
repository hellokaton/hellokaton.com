---
layout:     post
title:      "Spring Schema扩展"
category:    java
tags: ["spring"]
---

Spring从2.0开始引入了一个新的机制用于扩展xml模式，我们就可以编写自定义的xml bean解析器然后集成到 `Spring IoC` 容器中。

xml扩展大概有以下几个步骤：

- 编写自定义类
- 编写xml schema来描述自定义元素
- 编写`NamespaceHandler新样式.css`的实现类
- 编写`BeanDefinitionParser`实现类
- 把以上组建注册到Spring

<!-- more -->

## 编写自定义类

```java
public class Car {

	private String brand;
	private float engine;
	private int horsePower;
}
```

## 编写自定义schema
```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsd:schema xmlns="http://www.mycompany.com/schema/my"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema"
        xmlns:beans="http://www.springframework.org/schema/beans"
        targetNamespace="http://www.mycompany.com/schema/my"
        elementFormDefault="qualified"
        attributeFormDefault="unqualified">

    <xsd:import namespace="http://www.springframework.org/schema/beans"/>

    <xsd:element name="car">
        <xsd:complexType>
            <xsd:complexContent>
                <xsd:extension base="beans:identifiedType">
                    <xsd:attribute name="brand" type="xsd:string" use="required"/>
                    <xsd:attribute name="engine" type="xsd:float"/>
                    <xsd:attribute name="horsePower" type="xsd:int"/>
                </xsd:extension>
            </xsd:complexContent>
        </xsd:complexType>
    </xsd:element>
</xsd:schema>
```
然后我们就可以像这样定义car:
```xml
<my:car id="Ferrari458" brand="Ferrari" engine="4.5" horsePower="605"/>
```

## 编写NamespaceHandler
`NamespaceHandler`用于解析我们自定义名字空间下的所有元素，目前我们要解析上面的`my:car`元素。
`NamespaceHandler `里面只有3个方法：
- `init()`会在`NamespaceHandler`初始化的时候被调用。
- `BeanDefinition parse(Element, ParserContext)` - 当Spring遇到一个顶层元素的时候被调用。
- `BeanDefinitionHolder decorate(Node, BeanDefinitionHolder, ParserContext)` - 当Spring遇到一个属性或嵌套元素的时候调用.

Spring提供了默认实现类`NamespaceHandlerSupport`，我们只需在init的时候注册每个元素的解析器即可。

```java
public class CarNamespaceHandler extends NamespaceHandlerSupport {

	@Override
	public void init() {
		//遇到car元素的时候交给CarBeanDefinitionParser来解析
		registerBeanDefinitionParser("car", new CarBeanDefinitionParser());
	}
}
```

## 编写BeanDefinitionParser

当遇到car元素时，Spring会交给CarBeanDefinitionParser来解析。CarBeanDefinitionParser取出相应的属性然后设置到bean中。

```java
public class CarBeanDefinitionParser extends AbstractSingleBeanDefinitionParser {

	@Override
	protected Class<?> getBeanClass(Element element) {
		//car元素对应Car对象类型
		return Car.class;
	}

	@Override
	protected void doParse(Element element, BeanDefinitionBuilder builder) {
		
		String brand = element.getAttribute("brand");
		String engine = element.getAttribute("engine");
		String hp = element.getAttribute("horsePower");
		
		//把对应的属性设置到bean中
		if(StringUtils.hasText(brand))
			builder.addPropertyValue("brand", brand);
		
		if(StringUtils.hasText(engine))
			builder.addPropertyValue("engine", engine);

		if(StringUtils.hasText(hp))
			builder.addPropertyValue("horsePower", hp);
		
	}
}
```

## 注册handler和schema
为了让Spring在解析xml的时候能够感知到我们的自定义元素，我们需要把`namespaceHandler`和xsd文件放到2个指定的配置文件中，这2个文件都位于`META-INF`目录中。
#### META-INF/spring.handlers
`` `spring.handlers` ``文件包含了xml schema uri和handler类的映射关系，比如：
>http\://www.mycompany.com/schema/my=spring.xml.ext.schema.CarNamespaceHandler

这表示遇到`http://www.mycompany.com/schema/my`命名空间的时候会交给`CarNamespaceHandler`来处理。

注意上面的冒号转义。
key部分必须和xsd文件中的`targetNamespace`值保持一致。

#### META-INF/spring.schemas

>http\://www.mycompany.com/schema/my.xsd=META-INF/car.xsd


## 最后测试下

写个Spring配置文件：
```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans 	xmlns="http://www.springframework.org/schema/beans" 
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
		xmlns:my="http://www.mycompany.com/schema/my"
		xsi:schemaLocation="
	        http://www.springframework.org/schema/beans 
	        http://www.springframework.org/schema/beans/spring-beans.xsd
	        http://www.mycompany.com/schema/my 
	        http://www.mycompany.com/schema/my.xsd">

	<my:car id="Ferrari458" brand="Ferrari" engine="4.5" horsePower="605" />

</beans>

```

测试类：
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:app.xml" })
public class SchemaTest {

	@Autowired
	@Qualifier("Ferrari458")
	private Car car;

	@Test
	public void propertyTest() {
		assertNotNull(car);

		String brand = car.getBrand();
		float engine = car.getEngine();
		int horsePower = car.getHorsePower();

		assertEquals("Brand incorrect.Should be Ferrari.", "Ferrari", brand);
		assertEquals("Engine incorrect.Should be 4.5L.", 4.5, engine, 0.000001);
		assertEquals("HorsePower incorrect.Should be 605hp.", 605, horsePower);

	}
}

```

## 参考文献
- [Extensible XML authoring](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#xml-custom)
- [基于Spring可扩展Schema提供自定义配置支持](http://blog.csdn.net/cutesource/article/details/5864562)
