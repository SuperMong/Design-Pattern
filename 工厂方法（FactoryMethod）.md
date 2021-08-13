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

