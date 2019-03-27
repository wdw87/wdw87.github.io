---
title: '[ThinkingInJava]多态'
date: 2019-03-26 16:26:35
tags:
- Java基础
- 笔记
- Thinking in Java
---

## 8. 多态

### 8.1 绑定

> 将一个**方法调用**同一个**方法主体**关联起来被称为绑定

> 前期绑定：在程序执行前进行绑定
>
> 后期绑定（动态绑定）：在运行时根据对象的类型进行绑定

Java中除了static方法和final方法**（private方法属于final）**之外，其他方法都是动态绑定

#### 8.1.1 缺陷：“覆盖私有方法”

- 只有非**private**方法才能够被覆盖
- 基类中的private方法对于导出类既不可见，也不能重载

<!-- more -->

```java
public class PrivateOverride{
	private void f() {
		System.out.println("private f()");
	}
	public static void main(String[] args) {
		PrivateOverride po = new Derived();
		po.f();
	}
	
}
class Derived extends PrivateOverride{
	public void f() {    //此处没有发生覆盖，基类中的private方法既不可见，也不能重载
		System.out.println("public f()");
	}
}
sout:
private f()
```

#### 8.1.2缺陷：域与静态方法

- 只有普通方法的调用可以是多态的
- 静态方法没有多态
- 域的访问在编译期进行解析，不会发生多态

```java
class Car{
	public int id = 0;
	int getId() {
		return id;
	}
}

public class BenzCar extends Car{
	public int id = 1;
	int getId() {
		return id;
	}
	public static void main(String[] args) {
		Car benzCar = new BenzCar();
		System.out.println(benzCar.id);  //域不会发生多态
		System.out.println(benzCar.getId());
	}
}
sout：
0
1
```

### 8.2 构造器和多态

#### 8.2.1 构造器的调用顺序

- 构造器实际上是static方法，只不过是隐式声明的
- 构造器不具有多态
- 基类构造器总是在导出类构造器之前调用，因为构造器需要检查对象是否正确的被调用，导出类只能访问自己的成员，只有**构造器才有权限对自己的元素进行初始化**

#### 8.2.2 继承与清理

- 如果需要手动清理对象，必须从导出类开始，直到基类

#### 8.2.3 构造器内部的多态方法的行为

- 初始化的实际顺序为：
  - 为对象分配存储空间，存储空间1初始化为二进制0
  - 调用基类构造器
  - 按照声明的顺序调用成员的初始化方法
  - 调用导出类的构造器主体
- 如果在构造器中调用本对象的多态方法，方法仍然会发生多态，但这个方法所操作的成员**可能还未初始化**

### 8.3 向下转型

- Java中所有的转型都会得到检查，如果转型不成功，则会抛出一个**ClassCastExeception**。
- 这种在运行期间对类型进行检查的行为称为“**运行时类型识别(RTTI)**”

```java
class Useful{
	public void f() {}
}

class MoreUseful extends Useful{
	public void f() {}
	public void u() {}
}

public class RTTI {
	public static void main(String[] args) {
		Useful[] x = {
				new Useful(),
				new MoreUseful()
		};
		x[0].f();
		//x[1].u();  ERROR!!! 由于向上转型，丢失了导出类特有的方法
		((MoreUseful)x[1]).u();  //RTTI，向下转型
		((MoreUseful)x[0]).u();  //运行时抛出异常java.lang.ClassCastException
	}
}
```

