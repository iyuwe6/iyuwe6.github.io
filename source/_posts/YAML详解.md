---
title: YAML详解
date: 2018-10-16 17:28:07
tags: 
  - YAML
categories: 
  - YAML
---

### 简介
YAML是一种简洁的非标记语言(YAML Ain't Markup Language)，YAML以数据为中心，使用空白，缩进，分行组织数据，从而使得表示更加简洁易读，常用于作为配置文件，比JSON更加简洁。
YAML官网：http://yaml.org/
js-yaml: https://github.com/nodeca/js-yaml 
YAML转化JSON: http://nodeca.github.io/js-yaml/
<!--more-->
### YAML的设计目标
- 人类容易阅读
- 可用于不同程序间的数据交换
- 适合描述程序所使用的数据结构，特别是脚本语言
- 丰富的表达能力与可扩展性
- 易于使用

### YAML与XML和、JSON比较
YAML与XML：
- 具有XML同样的优点，但比XML更加简单、敏捷等；
  YAML与JSON：
- JSON可以看作是YAML的子集，也就是说JSON能够做的事情，YAML也能够做
- YAML能表示得比JSON更加简单和阅读，例如“字符串不需要引号”。所以YAML容可以写成JSON的格式，但并不建议这种做法
- YAML能够描述比JSON更加复杂的结构，例如“关系锚点”可以表示数据引用（如重复数据的引用）

### YAML的适用范围
- 由于实现简单，解析成本很低，YAML特别适合在脚本语言中使用。列一下现有的语言实现：Ruby，Java，Perl，Python，PHP，OCaml，JavaScript。除了Java，其他都是脚本语言.
- YAML比较适合做序列化。因为它是宿主语言数据类型直转的，由于兼容性问题，不同语言间的数据流转建议现在不要用YAML.
- YAML做配置文件也不错。比如Ruby on Rails的配置就选用的YAML。对ROR而言，这很自然，也很省事

### 语法
- 大小写敏感
- 使用缩进表示层级关系，禁止使用tab键缩进，只能使用空格键，建议使用两个空格，要保证相同层级的元素左对齐
- '#'表示注释，从这个字符到行尾，都会被解析器忽略
- 文档使用Unicode编码作为字符标准编码，例如UTF-8
- 字符串可以不用引号(不推荐这样)，也可以使用单引号或者双引号

### 书写格式
#### 多行缩进
数据结构可以用类似大纲的缩排方式呈现，结构通过缩进控制，连续的项目通过-来表示，Map结构里面的key/value对用冒号：来分隔。实例如下：
```yaml
house:
  family:
    name: Doe
    parents:
      - John
      - Jane
    children:
      - Paul
      - Mark
      - Simone
  address:
    number: 34
    street: Main Street
    city: Nowheretown
    zipcode: 12345
```

#### 单行缩写
YAML也可以用来描述好几行相同结构的数据的单行缩写语法，数组用[]包括起来，hash用｛｝包括起来。因此，可以将上面的文件改写为如下格式：
```yaml
house:
  family: { name: Doe, parents: [John, Jane], children: [Paul, Mark, Simone] }
  address: { number: 34, street: Main Street, city: Nowheretown, zipcode: 12345 }
```

### YAML组织结构
YAML文件可以由一个或者多个文档组成（也即相对独立的组织结构组成），文档间使用“---”（三个横线）在每文档开始作为分隔符。同时，文档也可以使用“...”（三个点号）作为结束符（可选）。如下图所示：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1o219xdo4j30es0h50t6.jpg)
注意：

- 如果只是单个文档，分隔符可省略；
- 每个文档并不需要使用结束符“...”来表示结束，但是对于网络传输或者流来说，作为明确结束的符号，有利于软件处理。（例如不需要知道流关闭就能知道文档结束）

### 数据结构
YAML支持的数据结构有以下三种：
- 对象：键值对的集合，又称为映射(mapping)/哈希(hash)/字典(dictionary)
- 数组：一组依次序排列的值，又称为序列(sequence)/列表(list)
- 纯量(scalars)：单个的、不可再分的值

#### 对象
对象的一组键值对，使用冒号结构表示。
```yaml
animal: pets
```
转为JSON:
```json
{ animal: 'pets' }
```
上面为多行缩写格式，当然也可以用单行缩写格式，如下：
```yaml
hash: { name: Steve, foo: bar }
```
转为JSON:
```json
{ hash: { name: 'Steve', foo: 'bar' } }
```

#### 数组
一组连词线开头的行，构成一个数组。
```yaml
- Cat
- Dog
- Goldfish
```
转为JSON:
```json
[ 'Cat', 'Dog', 'Goldfish' ]
```
数组也可以用单行缩写格式，如下：
```yaml
animal: [Cat, Dog]
```
转为JSON:
```json
{ animal: [ 'Cat', 'Dog' ] }
```

#### 纯量
纯量是最基本的、不可分割的值。以下数据类型都属于纯量。
    字符串、布尔值、整数、浮点数、Null、时间、日期
##### 整数和浮点数
直接以字面量的形式表示
```yaml
#YAML形式
number: 12.30
#JSON形式
{ number: 12.30 }
```
##### 布尔值
直接用true和false表示
```yaml
#YAML格式
isSet: true
#JSON格式
{ isSet: true }
```
##### NULL
用~表示
```yaml
#YAML格式
parent: ~ 
#JSON格式
{ parent: null }
```
##### 时间
采用ISO8681格式
```yaml
#YAML格式
iso8601: 2001-12-14t21:59:43.10-05:00 
#JSON格式
{ iso8601: new Date('2001-12-14t21:59:43.10-05:00') }
```
##### 日期
采用复合ISO8681格式的年月日
```yaml
#YAML格式
date: 1976-07-31
#JSON格式
{ date: new Date('1976-07-31') }
```
##### 强制转换数据类型
YAML允许使用两个感叹号!!，强制类型转换、
```yaml
#YAML格式
e: !!str 123
f: !!str true
#JSON格式
{ e: '123', f: 'true' }
```
##### 字符串
字符串是最常见，也是最复杂的一种数据类型。
- 字符串默认不使用引号表示。
```yaml
#YAML格式
str: 这是一行字符串
#JSON格式
{ str: '这是一行字符串' }
```
- 如果字符串中包含空格或者特殊字符，需要放在引号之中。
```yaml
#YAML格式
str: '内容： 字符串'
#JSON格式
{ str: '内容: 字符串' }
```
- 单引号和双引号都可以使用，双引号不会对特殊字符转义。
```yaml
#YAML格式
s1: '内容\n字符串'
s2: "内容\n字符串"
#JSON格式
{ s1: '内容\\n字符串', s2: '内容\n字符串' }
```
- 单引号之中如果还有单引号，必须连续使用两个单引号转义。
```yaml
#YAML格式
str: 'labor''s day'
#JSON格式
{ str: 'labor\'s day' }
```
- 字符串可以写成多行，从第二行开始，必须有一个单空格缩进。换行符会被转为空格。
```yaml
#YAML格式
str: 这是一段
  多行
  字符串
#JSON格式
{ str: '这是一段 多行 字符串' }
```
- 多行字符串可以使用|保留换行符，也可以使用>折叠换行。
```yaml
#YAML格式
this: |
  Foo
  Bar
that: >
  Foo
  Bar
#JSON格式
{ this: 'Foo\nBar\n', that: 'Foo Bar\n' }
```
- +表示保留文字块末尾的换行，-表示删除字符串末尾的换行。

```yaml
#YAML格式
s1: |
  Foo
  
s2: |+
  Foo
  
  
s3: |-
  Foo
  
```
- 字符串之中可以插入 HTML 标记。
```yaml
#YAML格式
message: |
  <p style="color: red">
    段落
  </p>
#JSON格式
{ message: '\n<p style="color: red">\n  段落\n</p>\n' }
```
##### 引用
锚点 & 和别名 * ，可以用来引用。

```yaml
#YAML格式
defaults: &defaults
  adapter:  postgres
  host:     localhost
 
development:
  database: myapp_development
  <<: *defaults
 
test:
  database: myapp_test
  <<: *defaults
#JSON格式
defaults:
  adapter:  postgres
  host:     localhost
 
development:
  database: myapp_development
  adapter:  postgres
  host:     localhost
 
test:
  database: myapp_test
  adapter:  postgres
  host:     localhost

```
说明： & 用来建立锚点（defaults )， <<表示合并到当前数据， * 用来引用锚点。