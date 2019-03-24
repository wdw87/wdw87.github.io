---
title: '[ThinkingInJava]复用类'
date: 2019-03-24 11:34:56
tags:
- Java基础
- 笔记
- Thinking in Java
---

## 7.复用类

### 7.1 组合

> 在类中new另外一个对象，以添加该对象的属性

```java
public class Computer {
    public Computer() {
        CPU cpu=new CPU();
        RAM ram=new RAM();
        Disk disk=new Disk();
    }
}
class CPU{    }
class RAM{    }
class Disk{    }
```



### 7.2 继承

> 从基类继承得到子类，得到该基类的特性

- 导出类包含基类
- Java会自动在导出类的构造器中插入对基类构造器的调用

```java
class Game{
	public Game() {
		System.out.println("Game constructure");
	}
}

class BoardGame extends Game{
	public BoardGame(){
		System.out.println("BoardGame constructure");
	}
}

public class Chess extends BoardGame{
	Chess(){
		System.out.println("Chess BoardGame constructure");
	}
	public static void main(String[] args) {
		Chess chess = new Chess();
	}
}
/*
sout：
Game constructure
BoardGame constructure
Chess BoardGame constructure
*/
```

- 如果基类只有带参数的构造器，没有默认的无参构造器，就必须用super显示的调用基类构造器

```java
class Game{
	public Game() {
		System.out.println("Game constructure");
	}
}

class BoardGame extends Game{
	public BoardGame(int i){
		System.out.println("BoardGame constructure");
	}
}

public class Chess extends BoardGame{
	Chess(){
		super(1); //此处显示调用基类构造器并传入参数！！
		System.out.println("Chess BoardGame constructure");
	}
	public static void main(String[] args) {
		Chess chess = new Chess();
	}
}
```

### 7.3 代理

> 在代理类中创建某功能的类，获得该类的**部分**属性

- 代理类的接口与底层类的接口相同，但是拥有更多控制力

```java
class SpaceShipControls{
	void up(int velocity) {}
	void down(int velocity) {}
	void left(int velocity) {}
	void right(int velocity) {}
	void foreord(int velocity) {}
	void back(int velocity) {}
	void turboBoost() {}
}

public class SpaceShip {
	private String name;
	private SpaceShipControls controls = new SpaceShipControls();
	public SpaceShip(String name) {
		this.name = name;
	}
	public void up(int velocity) {
		controls.up(velocity);
	}
	public void down(int velocity) {
		controls.down(velocity);
	}
	public void left(int velocity) {
		controls.left(velocity);
	}
	public void right(int velocity) {
		controls.right(velocity);
	}
	public void foreord(int velocity) {
		controls.foreord(velocity);
	}
	public void back(int velocity) {
		controls.back(velocity);
	}
}
```

### 7.4 名称屏蔽

- 导出类中定义的同名方法不会屏蔽其在积累中的任何版本
- 与C++不同，C++这样做需要屏蔽基类方法

```java
class Homer{
	void doh(char c) {
		System.out.println("doh(char c)");
	}
	void doh(int i) {
		System.out.println("doh(int i)");
	}
}
//Bart通过继承Homer，并给Homer添加了新的同名方法 doh(float f)
class Bart extends Homer{
	void doh(float f) {
		System.out.println("doh(float f)");
	}
}

public class Hide {
	public static void main(String[] args) {
		Bart b = new Bart();
        //三个方法都可以使用
		b.doh('c');
		b.doh(1);
		b.doh(1.0f);
	}
}
/*
sout：
doh(char c)
doh(int i)
doh(float f)
*/
```

- Java SE5新增了@Override注解，避免重载而非覆写方法

```java
class Bart extends Homer{
	@Override
	void doh(float f) { //添加@Override，在重载而非覆写方法时，编译器报错
		System.out.println("doh(float f)");
	}
}
```

### 7.5 protect关键字

- 提供包访问权限

### 7.6 向上转型

- 父类引用可以自动指向子类对象，但只能访问和调用到来自于父类的属性和行为
- 向上转型的过程中丢失了子类独有的方法

```java
class Car{
	public void run() {
		System.out.println("Car running");
	}
}

public class BenzCar extends Car{
	public void run() {
		System.out.println("BenzCar running");
	}
	public void price() {
		System.out.println("BenzCar price");
	}
	public static void main(String[] args) {
		Car car = new BenzCar();
		car.run();
		//car.price();  ERROR!!!
	}
}
```





### 7.7 final关键字

可能用到final的三种情况：数据、方法、类

#### final数据

- 编译常量：编译时完成，必须是**基本类型**，以final表示
- 既是static 又是final的域只占一段不能改变的存储空间
- 对于引用对象，**final使引用恒定不变**，但对象本身时可以修改的
- Java未提供使任何对象恒定不变的途经
- 根据惯例，既是static 又是final的域使用大写、下划线

```java
import java.util.Random;

class Value{
	int i;
	public Value(int i) {
		this.i = i;
	}
}

public class FinalData {
	//Can br compile-time constants:
	private final int valueOne = 9;
	private static final int VALUE_TWO = 99;
	//Typical public constant:
	public static final int VALUE_THREE = 99;
	//Cannot be compile-time constants:
	private final int int4 = new Random(47).nextInt(20);
	static final int INT_5 = new Random(47).nextInt(20);
	private Value v1 = new Value(11);
	private final Value v2 = new Value(22);
	private static final Value VAL_2 = new Value(33);
	//Arrays:
	private final int[] a = {1,2,3,4,5};
	public static void main(String[] args) {
		FinalData fd1 = new FinalData();
		//fd1.valueOne++;  ERROR
		fd1.v2.i++;
		fd1.v1 = new Value(9);
		for(int i = 0;i < fd1.a.length;i++) {
			fd1.a[i]++;
		}
		//fd1.v2 = new Value(0);  ERROR!
		//fd1.VAL_3 = new Value(1);  ERROr!!!
		//fd1.a = new int[3];   ERROr!!!
	}
}
```

- final参数
  - 基本类型可读不可改
  - 引用类型不可修改指向对象，但是可以修改指向对象的值

```java
import java.util.Random;

class Value{
	int i;
	public Value(int i) {
		this.i = i;
	}
}
public class FinalData {
	public void changeValue(final Value value) {  //final参数
        //value = new Value(2);   ERROR！！！！
		value.i++;
	}
	public static void main(String[] args) {
		FinalData fd1 = new FinalData();
		Value val1 = new Value(1);
		System.out.println(val1.i);
		fd1.changeValue(val1);
		System.out.println(val1.i);
	}

}
sout：
1
2

```

#### final方法

- 继承中时方法行为保持不变，并且不会被覆盖
- 类中的**private**方法都**隐式**的指定为**final**

#### final类

- 不允许继承该类
- **final**类中的所有方法都**隐式**的指定为**final**

### 7.8 初始化及类的加载

- Java中的类在初次使用时才加载
- 当访问**static**域或**static**方法时也会发生类的加载