---
title: 常用对象的API
date: 2019-12-18 21:27:37
tags: 
 - Java
 - 学习笔记
categories:
 - Java基础
comments: true
keywords: 
description:
---

***

### String类


* 概念
	程序中所有的双引号字符串,都是String类的对象

* 特点
  * 字符串的内容不可以被改变
  * 字符串可以被共享使用
  * 字符串效果上相当于char[]字符数组,底层原理是byte[]字节数组

* 创建字符串
  * 通过空参构造
	  String string = new String()
  * 通过字符串构造
	  String string =new String(String original)
  * 通过字符数组构造
	  String string = new String(char[] chars)
  * 通过字节数组构造
	  String string = new String(byte[] bytes)
  * 通过双引号直接创建
	  String string = "hello"
    * 注意事项
      1. 前三种通过new的创建方式,string变量指向的是堆中的地址
      2. 通过双引号直接创建,string变量指向的是常量池中的地址
* ==号
  * 比较基本类型	比较的是变量中具体的值

  * 比较引用类型	比较的是变量中的地址值
* **常用方法**
  1. 比较
      * boolean equals(Object obj)
	    比较两个字符串是否相等
      * boolean equalsIgnoreCase(String string)
	    比较两个字符串是否相等,忽略大小写
  2. 获取
      * int length()
		获取长度
      * char charAt(int index)
		获取指定索引位置的字符
      * int indexOf(String string)
		获取传入的字符串在当前字符串中第一次出现的位置,未出现返回-1
  3. 了解
      * 截取
        * String subString(int index)
			从指定位置开始截取,一直截取到最后
        * String subString(int begin,int end)
			从指定位置开始截取,截取到指定位置,含头不含尾
      * 转换
        * String replace(CharSequence oldString,CharSequence newString)
			将当前字符串中的指定内容替换为新的内容,将新字符串返回
        * String toLowerCase()
			将字符串转为小写
        * String toUpperCase()
			将字符串转为大写
        * String trim()
			去掉字符串两端的空格
      * 判断
        * boolean contains(CharSequence s)
			判断字符串是否包含指定的字符串
        * boolean startsWith(String prefix)
			判断字符串是否以指定的字符串开头
        * boolean endsWith(String suffix)
			判断字符串是否以指定的字符串结尾
					
***

### StringBuilder类


* 介绍
	一个可变的字符序列
* 构造方法
  * new StringBuilder()
	创建空的StringBuilder对象
  * new StringBuilder(String string)
	创建带有指定数据的StingBuilder对象
* 成员方法
  * StringBuilder append(...)
	将任意类型的数据追加到StringBuilder对象中
  * StringBuilder reverse()
	将StringBuilder对象中的数据反转
  * String toString()
	将StringBuilder对象中的数据转换为String返回
	
* **使用场景**
  * 1.大量的字符串拼接
  * 2.字符串反转

***

### Arrays类


* 介绍
	操作数组相关的工具类,里面有静态方法,可以通过类名进行调用
* 方法
  * String toString(数据类型[] 变量名)
	将数组中的内容以字符串的形式返回
  * void sort(数据类型[] 变量名)
	将数组中的内容进行排序

***

### ArrayList集合


* 介绍
	是一个长度可变的容器
* 数组和集合的区别
	1. 数组的长度是不可变的,集合的长度是可变的
	2. 数组既可以存储基本类型也可以存储引用类型,集合只能存储引用类型
* 泛型
	约束集合存储的数据类型, <> 中只能是引用数据类型
* 使用步骤
	1. 导包
		import java.util.ArrayList
	2. 创建对象
		ArrayList<引用数据类型> arrayList = new ArrayList<>();
	3. 使用
		arrayList.成员方法();
* 常用方法
  * 添加
    * boolean add(E e)
		将指定元素添加到集合的末尾
    * void add(int index, E element)  
		将指定元素添加到集合的指定位置
  * 删除
    * E remove(int index)
		移除指定位置的元素,返回值是被移除的元素
    * boolean remove(Object o)
		移除指定的元素,返回值是移除是否成功
  * 修改
    * E set(int index, E element)
		修改指定位置的元素,返回值是修改前的元素
  * 获取
    * E get(int index)
		获取指定位置的元素
  * 长度
    * int size()
		获取集合的长度
	
* 包装类
  * 基本类型对应包装类
		int	Integer
		char	Character
		byte	Byte
		short	Short
		long	Long
		float	Float
		double	Double
		boolean	Boolean
	
  * 装箱和拆箱
    * 装箱
		将基本类型数据变成包装类
    * 拆箱
		将包装类型数据变成基本类型
    * 注意
		JDK1.5之后,可以根据场景需求自动进行装箱和拆箱
		
***

### Math类


* 介绍
	操作数学相关的工具类,里面有静态方法,可以通过类名进行调用

* 方法
  * double abs(double num)		获取绝对值
  * double ceil(double num)		向上取整
  * double floor(double num)		向下取整
  * long round(double num)		四舍五入
  * double random()		获取一个0~1的随机数,左闭右开
  * int max(int a, int b)		获取两个数中的较大值
  * int min(int a, int b)		获取两个数中的较小值
  * double pow(double a, double b)		获取a的b次幂

* 成员变量
  * PI		圆周率近似值

***

### System类


* 常用方法
  * long currentTimeMillis()		获取当前系统时间毫秒值
		时间原点:	1970年1月1日 00:00:00
  * void exit(int status)		终止当前正在运行的虚拟机
		0:表示正常终止

***

### Object类


* 特点
  1. 所有类的父类(跟类,基类)
  2. 任意类都直接或间接继承Object类
  3. 任意类都可以直接使用Object类中的方法

* toString 方法
  * 重写前
	调用的是Object类的 toString 方法,返回的是全路径类名 + @ + 十六进制地址值
  * 重写后
	调用的是自身的 toString 方法,返回的属性名+属性值

* equals方法
  * ==号
	比较基本类型时,比较的是具体指
	比较引用类型时,比较的是地址值
  * 重写前
	调用的是Object类的equals方法,比较的是地址值
  * 重写后
	调用的是自身的equals方法,比较的是属性值
	
***

### Date类


* 毫秒值
	1秒 = 1000毫秒
	时间原点:	1970年1月1日 00:00:00
* 构造方法
  * new Date():
    创建当前系统时间的时间对象
  * new Date(long times):
    创建指定毫秒值的时间对象(在时间原点基础上加上指定的时间)
* 成员方法
  * long getTime()
	获取当前时间对象对应的毫秒值
  * void setTime(long time)
	将指定毫秒值设置给当前时间对象

***

### SimpleDateFormat类


* 构造方法
  * new SimpleDateFormat()
	创建默认模板的SimpleDateFormat对象
  
  * new SimpleDateFormat(String pattern)
	创建指定模板的SimpleDateFormat对象
		
		| y    | M    | d    | H    | m    | s    |
		| ---- | ---- | ---- | ---- | ---- | ---- |
		| 年   | 月   | 日   | 小时 | 分钟 | 秒   |


* 成员方法
  * String format(Date date)
	将指定的时间对象按照指定的模板格式化成字符串
  * Date parse(String string)
	将指定的字符串按照指定的模板解析为时间对象
* 注意事项
	通过parse方法将字符串解析为时间对象时,传入的字符串必须与指定模板匹配,否则出现ParseException异常

***

### Calendar类


* 获取对象
	Calendar calendar = Carlendar.getInstance()
* 成员方法
  * int get(int field)
	获取指定日历字段的值
  * void add(int field,int value)
	将指定日历字段的值增加或者减少指定值
  * void set(int year,int month,int date)
	设置当前日历对象的年,月,日
* 注意事项
	西方的月份周期是 0~11 , 获取时要进行 +1 ,才是我们的月份,设置时我们的月份 -1

***











