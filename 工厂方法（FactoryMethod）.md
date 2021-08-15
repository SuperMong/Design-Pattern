# 工厂方法（FactoryMethod）

### 意图

定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method使一个类的实例化延迟到其子类

### 别名

虚构造器（virtual constructor）

### 适用性

- 当一个类不知道它所必须创建的对象的类的时候
- 当一个类希望由它的子类来指定它所创建的对象的时候
- 当类将创建对象的职责委托给多个帮助子类中的某一个，并且你希望将哪一个帮助子类是代理者这一信息局部化的时候

### 结构

![avatar](/image/工厂方法结构图.png)

### 参与者

- Product：定义工厂方法所创建的对象的接口
- ConcreteProduct：实现Product接口
- Creator：
  - 声明工厂方法，该方法返回一个Product类型的对象。Creator也可以定义一个工厂方法的缺省实现，它返回一个缺省的ConcreteProduct对象
  - 可以调用工厂方法创建一个Product对象
- ConcreteCreator：重定义工厂方法以返回一个ConcreteProduct实例

### 协作

Creator依赖于它的子类来定义工厂方法，所以它返回一个适当的ConcreteProduct实例

### 效果

优点：

- 代码仅处理Product接口，因此可以与任何ConcreteProduct类一起使用
- 为子类提供钩子（hook）：用工厂方法在一个类的内部创建对象通常比直接创建对象更灵活
- 连接平行的类层次：工厂方法不一定是被Creator调用，将一个类把它的一些职责委托给另一个独立的类的时候，就产生平行类层次，委托的类可以调用工厂方法创建被委托的类的对象

缺点：

- 可能会为了创建一个特定的ConcreteProduct对象不得不创建Creator子类

### 实例

有长袖和短袖两种衣服，保暖程度不同：

```c++
class Cloth {
public:
    virtual int KeepWarm() const = 0;
};
```

```c++
class LongSleeve : public Cloth {
public:
	LongSleeve();
    virtual int KeepWarm() const
    	{ return 10; }
};
```

```c++
class ShortSleeve : public Cloth {
public:
	ShortSleeve();
	virtual int KeepWarm() const
		{ return 5; }
};
```

纺织品加工厂根据季节选择生产哪一种：

```c++
class Creator {
public:
    // AnOperation
	Cloth* GetCloth();
protected:
    // FactoryMethod
	virtual Cloth* CreateCloth() const = 0;
private:
	Cloth* _cloth;
};

Cloth* Creator::GetCloth() {
	if (_cloth = nullptr) {
		_cloth = CreateCloth();
	}
	return _cloth;
}
```

```c++
class LongCreator : public Creator {
public:
    LongCreator();
    
    virtual Cloth* CreatCloth()
    	{ return new LongSeleeve; }
};
```

```c++
class ShortCreator : public Creator {
public:
    ShortCreator();
    
    virtual Cloth* CreatorCloth()
    	{ return new ShortSeleeve; }
};
```

这样在纺织品加工厂生产衣服时候，很容易选择具体生产长袖还是短袖：

```c++
// 生产长袖
LongCreator lc;
LongSleeve* ls = lc.GetCloth();

// 生产短袖
ShortCreator = sc;
ShortSleeve* ss = sc.GetCloth();
```

### 技巧

1. 主要情况分两种：

   - Creator是一个抽象类并且不提供它所声明的工厂方法的实现：避免了不得不实例化不可预见子类的问题
   - Creator是一个具体的类型且为工厂方法提供一个缺省的实现：为了灵活性

2. 参数化工厂方法：采用一个表示要被创建的对象的参数，让工厂方法可以创建多种产品。

   ```C++
   class Creator {
   public:
       virtual Product* Creat(ProductId id);
   };
   
   Product* Creator::Creat(ProductId id) {
       if (id == A) return new ProductA;
       if (id == B) return new ProductB;
       // ...
       return nullptr;
   };
   ```

   可以通过重定义工厂方法进行扩展或者改变Creator生产的产品：

   ```c++
   Product* OtherCreator::Creat(ProductId id) {
       if (id == B) return new ProductA;
       if (id == A) return new ProductB;
       
       return Creator::Creat(id);
   }
   ```

3. 使用模板避免创建子类：使用Creator模板可以减少对子类的创建

4. 命名约定：使用统一的命名约定说明在使用工厂方法
