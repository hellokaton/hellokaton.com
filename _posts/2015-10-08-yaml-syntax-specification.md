---
layout: post
title: yaml语法规范
categories: java
tags: yaml
---

做 java 项目用的最多的配置文件就是 properites 或者 xml， xml 确实是被用烂了，Struts, Spring, Hibernate(ssh) 无一不用到 xml。相比厚重的 xml， properites 要清爽许多，一般的项目自己需要的配置也足够使用。但 properties 只支持 key=value 这种形式的配置，如果再遇到复杂结构的配置，恐怕难以胜任。

<!-- more -->

这时候 [YAML][1] 出场，yaml 不仅可以做到 properites 的小清新，也可以做到 xml 的表达复杂的结构的能以。

map

```bash
name: bastengao
money: 500W
interest: coding
```

array/collection

```bash
- 张三
- 李四
- 王武
```

<!--more-->

混合

```bash
- name: bastengao
  money: 500W
  interest: coding

- name: 张三
  money: 0.01
  interest: eating
```

以上只是举几个例子，详细的语法参考，[yaml spec 1.2][2]。

用 java 读取 yaml 文件可以使用 [snakeymal][3]，maven 项目可以直接引依赖。

```xml
<dependencies>
  ...
  <dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.13</version>
  </dependency>
  ...
</dependencies>
```

snakeyaml 读取 yaml

```java
InputStream input = new FileInputStream("config.yml");
Yaml yaml = new Yaml();
Map<String, Object> object = (Map<String, Object>) yaml.load(input);
```

  [1]: http://www.yaml.org/
  [2]: http://www.yaml.org/spec/1.2/spec.html
  [3]: https://code.google.com/p/snakeyaml/