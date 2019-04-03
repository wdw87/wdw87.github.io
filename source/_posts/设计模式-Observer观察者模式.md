---
title: '[设计模式]Observer观察者模式'
date: 2019-04-03 23:13:39
tags:
- 设计模式
---

## Observer 观察者模式

> 观察者模式定义了对象之间一对多依赖，当一个对象改变状态时，它所有的依赖者都会受到通知并且更新

> **简单地说：你订阅公众号，公众号为主题，你是观察者，公众号有了新动态你立马收到通知，取消订阅后就不会收到通知了**

**设计原则**

- 找出程序中会变化的方面，然后将其和固定不变的方面分离
- 针对接口编程，而不是针对实现编程
- 多用组合，少用继承

### 1. 示例结构图

以HeadFirst书中的气象站为例类图如下（代码在最后）：

<img src = "http://www.plantuml.com/plantuml/png/dPB1JiCm38RFzLFak0bfBm27IHjSEUp0sKqzC4gJAd4gJQNlJe5cQZFQmswL_y__xyRU1q4liJR0LiYVKPCwScWCNfuDrMIbWzPnfQg_ucOB_GHzBGFvblm8nQP2eOmvnVAJzE1J_3AUtZaCMchTf0_bjezNfdOjhH7M2PylLsAezm1ZqjFNRlT6A1_aZoYe864_mY5wJrOEpbOix6mO_tO6lJXF6eFyfvq4XOEmJfqAvW-scfAZULPEB2I2DXY2Mehfe7XeOcbOg_dor4J85ZPQPkDq2mrB0ODHSheT-mwpqMkopjnb_Q0oDZki5VBsm0heOK-sYhNv1W00"/>

<!--more-->

 气象站，发布气象数据,为主题

```java
WeatherData 
```

布告栏,显示气象数据，为观察者

```java
CurrentConditionsDisplay
ForecastDisplay
StatisticsDisplay 
```

### 2. 小结

- JDK中观察者通过一个类实现，限制了他的复用能力Java 9后废弃
- 观察者和被观察者是抽象耦合的

### 3. 示例代码

```java
/**
*主题接口
*声明了添加、删除、通知观察者的方法
*
*/
public interface Subject {
	public void registerObserver(Observer o);
	public void removeObserver(Observer o);
	public void notifyObserver();
}

```

```java
/**
*观察者接口
*声明了update()方法，当主题调用notifyObserver()方法时，观察者的update()方法被调用
*
*/
public interface Observer {
	public void update(float temp,float humidity,float pressure);
}
```

```java
/**
*显示接口
*与观察者设计模式无关，用于为观察者提供显示数据的接口
*
*/
public interface DisplayElement {
	public void display();
}
```

接口的实现类：

```java
import java.util.ArrayList;
/**
*主题的实现类
*
*/
public class WeatherData implements Subject{
	private float temperature;
	private float humidity;
	private float pressure;
    //ArrayList用于存放订阅了该主题的观察者，面向接口编程
	private ArrayList<Observer> observers;
	
	public WeatherData() {
		observers = new ArrayList<Observer>();
	}
	
	public void registerObserver(Observer o) {
		observers.add(o);
	}

	public void removeObserver(Observer o) {
		int i = observers.indexOf(o);
		if(i >= 0) {
			observers.remove(i);
		}
	}

	public void notifyObserver() {
		for(int i = 0; i<observers.size();i++) {
            //这里调用了观察者的update()方法，多态
			observers.get(i).update(temperature, humidity, pressure);
		}
	}
	
	public void measurementsChanged() {
		notifyObserver();
	}
	//这个函数用于模拟气象站接收到了新的气象数据
	public void setMeasurements(float temp,float humidity,float pressure) {
		this.temperature = temp;
		this.humidity = humidity;
		this.pressure = pressure;
		measurementsChanged();
	}
	
}
```

```java
/**
*观察者的实现类
*实现了Observer接口
*
*/
public class CurrentConditionsDisplay implements Observer,DisplayElement{
	
	private float temperature;
	private float humidity;
	private Subject subject;
	//在构造函数中将自己添加到主题中
	public CurrentConditionsDisplay(Subject s) {
		this.subject = s;
		this.subject.registerObserver(this);
	}
	
	public void display() {
		System.out.println("Current conditions: " + temperature + "F degrees and " + humidity + "% humidity");
	}

	public void update(float temp, float humidity, float pressure) {
		this.temperature = temp;
		this.humidity = humidity;
		display();
	}

}
```

测试类：

```java
public class WeathetStation {
	public static void main(String[] args) {
		WeatherData weatherData = new WeatherData();
		
		CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay(weatherData);
		
		weatherData.setMeasurements(80, 65, 30.4f);
		weatherData.setMeasurements(85, 60, 31.1f);
		weatherData.setMeasurements(75, 53, 28.5f);
		
	}
}
```

