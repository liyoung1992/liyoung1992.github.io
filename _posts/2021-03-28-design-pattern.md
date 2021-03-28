---
layout: post
title:  "常用设计模式总结"
categories: 设计模式
tags:  设计模式  C++ 
---


* content
{:toc}

常用设计模式总结。10种常用的设计模式。

<!--excerpt-->

## 概述

复习一下常用的设计模式，使用C++实现。常用的模式如下：

- 单例模式
- 简单工厂模式
- 工厂方法模式
- 抽象工厂模式
- 建造者模式
- 适配器模式
- 命令模式
- 观察者模式
- 策略模式

## 简单工厂模式

简单工厂模式，一个工厂，根据类别来生成具体的产品

```cpp
#include <iostream>
/*
简单工厂模式，一个工厂，根据类别来生成具体的产品
*/
class AbstractProduct_SF {
public:
	AbstractProduct_SF() = default;
	virtual ~AbstractProduct_SF() {};
	virtual void show() = 0;
};

class Product1SF : public AbstractProduct_SF {
public:
	void show() override{
		std::cout << "product 1" << std::endl;
	}
};
class Product2SF : public AbstractProduct_SF {
public:
	void show() override {
		std::cout << "product 2" << std::endl;
	}
};

class Factory {
public:
	AbstractProduct_SF* createProduct(int type) {
		if (type == 0) {
			return new Product1SF();
		}
		else {
			return new Product2SF();
		}
	}
};
```

## 工厂方法模式

工厂方法模式，不同的工厂生产不同的商品

```cpp
#include <iostream>
/*
工厂方法模式，不同的工厂生产不同的商品
*/

class FMProduct {
public:
	virtual ~FMProduct() {};
	virtual void show() = 0;
};

class FMProduct1 : public FMProduct {
public:
	void show() override {
		std::cout << "product 1" << std::endl;
	}
};
class FMProduct2 : public FMProduct {
public:
	void show() override {
		std::cout << "product 2" << std::endl;
	}
};

class FMFactory {
public:
	virtual ~FMFactory() {}
	virtual FMProduct* createProduct() = 0;
};

class FMFactory1 :public FMFactory{
public:
	FMProduct* createProduct() {
		return new FMProduct1();
	}
};

class FMFactory2 : public FMFactory {
public:
	FMProduct* createProduct() {
		return new FMProduct2();
	}
};


```

## 抽象工厂模式

抽象工厂模式是一种创建型设计模式， 它能创建一系列相关的对象， 而无需指定其具体类。

假设你正在开发一款家具商店模拟器。 你的代码中包括一些类， 用于表示：

1.一系列相关产品， 例如 椅子Chair 、 ​ 沙发Sofa和 咖啡桌Coffee­Table 。

2.系列产品的不同变体。 例如， 你可以使用 现代Modern 、 
​ 维多利亚Victorian 、 ​ 装饰风艺术Art­Deco等风格生成 椅子 、 ​ 沙发和 咖啡桌 。

```cpp
#include <iostream>
/*
抽象工厂模式是一种创建型设计模式， 它能创建一系列相关的对象， 而无需指定其具体类。

假设你正在开发一款家具商店模拟器。 你的代码中包括一些类， 用于表示：

1.一系列相关产品， 例如 椅子Chair 、 ​ 沙发Sofa和 咖啡桌Coffee­Table 。

2.系列产品的不同变体。 例如， 你可以使用 现代Modern 、 
​ 维多利亚Victorian 、 ​ 装饰风艺术Art­Deco等风格生成 椅子 、 ​ 沙发和 咖啡桌 。
*/
/*
抽象产品A
*/
class AbstractProductA {
public:
	AbstractProductA() = default;
	virtual ~AbstractProductA() {};
	virtual void createProductA() = 0;
};
class ConcreteProductA1 : public AbstractProductA {
public:
	virtual void createProductA() override {
		std::cout << "concrete product A1" << std::endl;
	};
};
class ConcreteProductA2 : public AbstractProductA {
public:
	virtual void createProductA() override {
		std::cout << "concrete product A2" << std::endl;
	};
};

/*
抽象产品B
*/
class AbstractProductB {
public:
	AbstractProductB() = default;
	virtual ~AbstractProductB() {};
	virtual void createProductB() = 0;
};
class ConcreteProductB1 : public AbstractProductB {
public:
	virtual void createProductB() override {
		std::cout << "concrete product B1" << std::endl;
	};
};
class ConcreteProductB2 : public AbstractProductB {
public:
	virtual void createProductB() override {
		std::cout << "concrete product B2" << std::endl;
	};
};

/*
工厂类
*/
class AbstractFactory
{
public:
	AbstractFactory() = default;
	virtual ~AbstractFactory() {};

	virtual AbstractProductA* createProductA() const = 0;
	virtual AbstractProductB* createProductB() const = 0;
};

class ConcreteFactory1 : public AbstractFactory {
public:
	ConcreteFactory1() = default;
	virtual ~ConcreteFactory1() {};
	AbstractProductA* createProductA() const override {
		std::cout << "factory1 create concrete A1" << std::endl;
		return new ConcreteProductA1();
	}
	AbstractProductB* createProductB() const override {
		std::cout << "factory1 create concrete B1" << std::endl;
		return new ConcreteProductB1();
	}
};
class ConcreteFactory2 : public AbstractFactory {
public:
	ConcreteFactory2() = default;
	virtual ~ConcreteFactory2() {};
	AbstractProductA* createProductA() const override {
		std::cout << "factory1 create concrete A2" << std::endl;
		return new ConcreteProductA2();
	}
	AbstractProductB* createProductB() const override {
		std::cout << "factory1 create concrete B2" << std::endl;
		return new ConcreteProductB2();
	}
};

```

## 建造者模式
与其他创建型模式不同， 生成器不要求产品拥有通用接口。
这使得用相同的创建过程生成不同的产品成为可能。

使用示例： 生成器模式是 C++ 世界中的一个著名模式。 
当你需要创建一个可能有许多配置选项的对象时， 该模式会特别有用。

识别方法： 生成器模式可以通过类来识别， 
它拥有一个构建方法和多个配置结果对象的方法。
生成器方法通常支持方法链 （例如 someBuilder->setValueA(1)->setValueB(2)->create() ）。

```cpp

#pragma once
#include <iostream>
#include <string>
#include <vector>
/*
* 
* 与其他创建型模式不同， 生成器不要求产品拥有通用接口。
这使得用相同的创建过程生成不同的产品成为可能。

使用示例： 生成器模式是 C++ 世界中的一个著名模式。 
当你需要创建一个可能有许多配置选项的对象时， 该模式会特别有用。

识别方法： 生成器模式可以通过类来识别， 
它拥有一个构建方法和多个配置结果对象的方法。
生成器方法通常支持方法链 （例如 someBuilder->setValueA(1)->setValueB(2)->create() ）。
*/

class Product1 {
public:
	std::vector<std::string> product_setting;
	void show() {
		std::vector<std::string>::iterator item;
		for (item = product_setting.begin(); 
			item != product_setting.end();++item)
		{
			std::cout << "item  " << *item << std::endl;
		}
		return;
	}
};
class Builder
{
public:
	virtual ~Builder() {};
	virtual void productPartA() const = 0;
	virtual void productPartB() const = 0;
	virtual void productPartC() const = 0;

};

class ConcreteBuilder : public Builder {
private:
	Product1* product;
public:
	ConcreteBuilder() {
		this->reset();
	}
	virtual ~ConcreteBuilder() { delete product; };
	void reset() {
		this->product = new Product1();
	}
	void productPartA() const override {
		this->product->product_setting.push_back("part A");
	}
	void productPartB() const override {
		this->product->product_setting.push_back("part B");
	}
	void productPartC() const override {
		this->product->product_setting.push_back("part C");
	}
	Product1* getProduct() {
		Product1* result = this->product;
		reset();
		return result;
	}
};

class Director {
private:
	Builder* builder;
public:
	void set_build(Builder* builder) {
		this->builder = builder;
	}
	void buildProductPartA() {
		builder->productPartA();
	}
	void buildProductFullPart() {
		builder->productPartA();
		builder->productPartB();
		builder->productPartC();
	}
};

```

## 适配器模式
适配器是一种结构型设计模式， 它能使不兼容的对象能够相互合作。

适配器可担任两个对象间的封装器， 
它会接收对于一个对象的调用， 并将其转换为另一个对象可识别的格式和接口。

```cpp
#pragma once
#include <iostream>
/*
适配器模式

适配器是一种结构型设计模式， 它能使不兼容的对象能够相互合作。

适配器可担任两个对象间的封装器， 
它会接收对于一个对象的调用， 并将其转换为另一个对象可识别的格式和接口。
*/
class Target {
public:
	virtual ~Target() = default;
	virtual void request() {
		std::cout << "target request" << std::endl;
	};
};

class Adaptee {
public:
	void new_request() {
		std::cout << "new request " << std::endl;
	}
};

class Adapter : public Target {
public:
	Adaptee* adaptee_;
	Adapter(Adaptee* ad) :adaptee_(ad) {}

	void request() override {
		adaptee_->new_request();
	}
};

```

## 命令模式
命令是一种行为设计模式， 它可将请求或简单操作转换为一个对象。

此类转换让你能够延迟进行或远程执行请求， 还可将其放入队列中。

```cpp

#pragma once
#include <iostream>
/**
 * 命令是一种行为设计模式， 它可将请求或简单操作转换为一个对象。

*  此类转换让你能够延迟进行或远程执行请求， 还可将其放入队列中。
 */
class Command {
public:
	virtual ~Command() = default;
	virtual void execute() const = 0;
};

class CopyCommand : public Command {
public:
	void execute() const override{
		std::cout << "execute copy" << std::endl;
	}
};

class PasteReceiver {
public:
	void doPaste() {
		std::cout << "do paste" << std::endl;
	}
};

//由具体的receiver来执行具体的业务逻辑
class PasteCommand : public Command {
public:
	PasteCommand(PasteReceiver* receiver) {
		receiver_ = receiver;
	}
	void execute() const override {
		receiver_->doPaste();
	}
private:
	PasteReceiver* receiver_;
};



class Invoker {
private:
	Command* copy_command_ = nullptr;
	Command* paste_command_ = nullptr;
public:
	void setCopyCmd(Command* cmd) {
		copy_command_ = cmd;
	}
	void setPasteCmd(Command* cmd) {
		paste_command_ = cmd;
	}
	void doCmd() {
		if (copy_command_) {
			copy_command_->execute();
		}
		if (paste_command_) {
			paste_command_->execute();
		}
	}

};

```

## 观察者模式
观察者模式是一种行为设计模式，
允许你定义一种订阅机制， 可在对象事件
发生时通知多个 “观察” 该对象的其他对象。

```cpp

#pragma once
#include <iostream>
#include <string>
#include <vector>
#include <list>

/*
观察者模式是一种行为设计模式，
允许你定义一种订阅机制， 可在对象事件
发生时通知多个 “观察” 该对象的其他对象。
*/

class Observer {
public:
	virtual ~Observer() = default;
	virtual void update(std::string msg) = 0;
};



class Subject {
public:
	virtual ~Subject() = default;
	virtual void attach(Observer* observer) = 0;
	virtual void detach(Observer* observer) = 0;
	virtual void notify() = 0;
};

class ConcreteSubject : public Subject {
public:
	virtual ~ConcreteSubject() = default;
	void attach(Observer* observer) override {
		observer_list_.push_back(observer);
	}
	void detach(Observer* observer) override {
		observer_list_.remove(observer);
	}
	void notify() override {
		std::list<Observer*>::iterator iterator 
			= observer_list_.begin();
		while (iterator != observer_list_.end())
		{
			(*iterator)->update(curr_msg_);
			++iterator;
		}
	}
	void updateMsg(std::string msg) {
		curr_msg_ = msg;
		notify();
	}
public:
	std::string curr_msg_;
	std::list<Observer*> observer_list_;
};


class ConcreteObserver :public Observer {
public:
	ConcreteObserver(Subject* subject)
		: subject_(subject) {
		this->subject_->attach(this);
	};
	void update(std::string msg) override {
		message_from_subject_ = msg;
		std::cout << "get update msg:" 
			<< msg << std::endl;
	}
	void removeFromSubject() {
		this->subject_->detach(this);
	}
	void show() {
		std::cout << "msg:" << message_from_subject_;
	}
private:
	Subject* subject_;
	std::string message_from_subject_;
};

```

## 策略模式

策略是一种行为设计模式， 它将一组行为转换为对象， 并使其在原始上下文对象内部能够相互替换。

原始对象被称为上下文， 它包含指向策略对象
的引用并将执行行为的任务分派给策略对象。 
为了改变上下文完成其工作的方式， 
其他对象可以使用另一个对象来替换当前链接的
策略对象。

```cpp

#pragma once
#include <iostream>

/*
策略是一种行为设计模式， 它将一组行为转换为对象， 并使其在原始上下文对象内部能够相互替换。

原始对象被称为上下文， 它包含指向策略对象
的引用并将执行行为的任务分派给策略对象。 
为了改变上下文完成其工作的方式， 
其他对象可以使用另一个对象来替换当前链接的
策略对象。
*/
class Strategy {
public:
	virtual ~Strategy() = default;
	virtual void alg() = 0;
};

class Strategy1 : public Strategy {
public:
	void alg() override {
		std::cout << "use strategy 1" << std::endl;
	}
};

class Strategy2 : public Strategy {
public:
	void alg() override {
		std::cout << "use strategy 2" << std::endl;
	}
};
class Context {
public:
	Context(Strategy* strategy) : strategy_(strategy) {

	}
	void set_strategy(Strategy* strategy) {
		if (strategy_) {
			delete strategy_;
		}
		this->strategy_ = strategy;
	}

	void use() {
		this->strategy_->alg();
	}
private:
	Strategy* strategy_;
};
```

## 使用方法

```cpp
#include <iostream>
#include "abstract_factory.h"
#include "builder.h"
#include "sample_factory.h"
#include "factory_method.h"
#include "Adapter.h"
#include "command.h"
#include "observer.h"
#include "strategy.h"
void abstractFactoryClientCode(const AbstractFactory& factory) {
	const AbstractProductA* product_a = factory.createProductA();
	const AbstractProductB* product_b = factory.createProductB();
	//TODO use
	delete product_a;
	delete product_b;
}


void testAbstractFactory() {
	ConcreteFactory1* f1 = new ConcreteFactory1();
	abstractFactoryClientCode(*f1);
	delete f1;
	std::cout << std::endl;
	ConcreteFactory2* f2 = new ConcreteFactory2();
	abstractFactoryClientCode(*f2);
	delete f2;
	return ;
}

void testBuilder(){
	Director* director = new Director();
	ConcreteBuilder* builder = new ConcreteBuilder();
	director->set_build(builder);
	std::cout << "part A" << std::endl;
	director->buildProductPartA();
	builder->getProduct()->show();
	std::cout << "full part" << std::endl;
	director->buildProductFullPart();
	builder->getProduct()->show();
}

void testSampleFactory() {
	Factory* sample_factory = new Factory();
	auto product1 = sample_factory->createProduct(0);
	product1->show();
	auto product2 = sample_factory->createProduct(1);
	product2->show();
	delete product1;
	delete product2;

	delete sample_factory;
}

void testFactoryMethod() {
	FMFactory1* factory1 = new FMFactory1();
	auto product1 = factory1->createProduct();
	product1->show();


	FMFactory2* factory2 = new FMFactory2();
	auto product2 = factory2->createProduct();
	product2->show();
}

void testAdapter() {
	Target* target = new Target();
	target->request();

	Adaptee* adaptee = new Adaptee();
	adaptee->new_request();

	Adapter* adapter = new Adapter(adaptee);
	adapter->request();
}

void testCommand() {
	Invoker* invoker = new Invoker();
	invoker->setCopyCmd(new CopyCommand());

	PasteReceiver* receiver = new PasteReceiver();
	invoker->setPasteCmd(new PasteCommand(receiver));

	invoker->doCmd();
}

void testObserver() {
	ConcreteSubject* subject = new ConcreteSubject();
	ConcreteObserver* ob1 = new ConcreteObserver(subject);
	ConcreteObserver* ob2 = new ConcreteObserver(subject);
	ConcreteObserver* ob3 = new ConcreteObserver(subject);
	
	subject->updateMsg("123");

	subject->updateMsg("456");
}
void testStrategy() {
	Context* context = new Context(new Strategy1());
	context->use();
	context->set_strategy(new Strategy2());
	context->use();
}
int main()
{
    std::cout << "Hello World!\n";
	//abstract factory
	testAbstractFactory();
	//builder
	testBuilder();
	//sample factory 
	testSampleFactory();

	//factory method
	testFactoryMethod();

	//adapter
	testAdapter();

	//command
	testCommand();

	//observer
	testObserver();

	//strategy
	testStrategy();
	
	system("pause");
}



```

