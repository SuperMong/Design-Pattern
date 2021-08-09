# 抽象工厂（Abstract Factory）

### 意图

提供一个接口以创建一系列相关或相互依赖的对象，而无需指定它们具体的类

### 别名

Kit

### 适用性

- 一个系统要独立于它的产品的创建、组合和表示
- 一个系统要由多个产品系列中的一个来配置
- 要强调一系列相关的产品对象的设计以便进行联合使用
- 提供一个产品类库，但只想显示它们的接口而不是实现

### 结构

![抽象工厂结构图](\image\抽象工厂结构图.png)

### 参与者

- AbstractFactory：声明一个创建抽象对象的操作接口
- ConcreteFactory：实现创建具体产品对象的操作
- AbstractProduct：为一类产品对象声明一个接口
- ConcreteProduct：定义一个将被相应的具体工厂创建的产品对象；实现AbstractProduct接口
- Client：仅使用由AbstractFactory和AbstractProduct类声明的接口

### 协作

- AbstractFactory将产品对象的创建延迟到它的ConcreteFactory子类
- 在运行时创建一个ConcreteFactory类的实例，用它创建具有特定实现的产品对象。客户通过不同的具体工厂创建不同的产品对象。

### 效果

- 优点：
  1. 分离的具体的类：将客户与类的实现分离，客户通过抽象接口操纵实例
  2. 使得易于更换产品系列：只需要改变具体工厂就能改变所有产品系列
  3. 有利于产品的一致性：当一个系列中的产品对象被设计成一起工作时，一次只能使用同一个系列中的对象，AbstractFactory很容易实现这一点
- 缺点：
  1. 难以扩展：添加一个新产品意味着要修改所有具体工厂

### 实现

考虑我们要生产手机和电脑组成的物联网设备，有华为和苹果两种品牌可以选择。

首先声明一个抽象类ElectronicFactory，它声明了创建抽象对象Computer和Phone的操作接口：

```c++
class ElectronicFactory {
public:
    ElectronicFactory();
    
    virtual Computer* CreateComputer() const = 0;
    virtual Phone* CreatePhone() const = 0;
}
```

接着声明ElectronicFactory的具体子类HuaweiFactory和AppleFactory，它们实现创建Computer和Phone的操作：

```c++
class HuaweiFactory : public ElectronicFactory {
public:
    HuaweiFactory();
    
    virtual Computer* CreateComputer() const
    	{ return new HuaweiComputer; }
    virtual Phone* CreatePhone() const
    	{ return new HuaweiPhone; }
}
```

```c++
class AppleFactory : public ElectronicFactory {
public:
    AppleFactory();
    
    virtual Computer* CreateComputer() const
    	{ return new AppleComputer; }
    virtual Phone* CreatePhone() const
    	{ return new ApplePhone; }
}
```

这样，客户可以通过把ElectronicFactory作为一个参数来创建Computer对象和Phone对象：

```c++
Equipment* Equipment::CreateEquipment(ElectronicFactory &factory) {
    Equipment* e = new Equipment;
    e->computer = factory.CreateComputer();
    e->phone = factory.CreatePhone();
    return e;
}
```

传入的ElectronicFactory参数不同，得到的设备品牌也不同：

```c++
HuaweiFactory hf;
Equipment* he = CreatEquipment(&hf); // 得到华为物联网设备

AppleFactory af;
Equipment* ae = CreatEquipment(&af); // 得到苹果物联网设备
```

### 技巧

下面是一些实现AbstractFactory的一些有用技术：

1. 将工厂作为单件：一个应用中一般每个产品系列只需要一个ConcreteFactory的实例，因此**工厂通常最好实现为一个Singleton**
2. 创建产品：AbstractFactory仅声明一个创建产品的接口，真正创建产品是由ConcreteFactory实现的。**通常为每个产品定义一个工厂方法**，一个具体的工厂为每个产品重定义该工厂方法，这种方法要求每个产品系列都有一个新的具体工厂子类，即使这些产品系列的差别很小。**如果有多个可能的产品系列，具体工厂也可以用Prototype模式实现**
3. 定义可扩展的工厂：**给创建对象的操作增加一个参数，指定被创建的对象的种类**。缺点在于所有类型都返回类型所给定的相同的抽象接口给客户，客户不能区分或对产品类别进行安全的假定。如果客户需要进行特定子类相关的操作，这些操作不能通过抽象接口得到