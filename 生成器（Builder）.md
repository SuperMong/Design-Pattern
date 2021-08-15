# 生成器（Builder）

### 意图

将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示

### 适用性

- 当创建复杂对象的算法应该独立于该对象的组成部分以及它们的装配方式
- 当构造过程必须允许被构造的对象有不同的表示

### 结构

![avatar](/image/生成器结构图.png)

### 参与者

- Buidler：为创建一个Product对象的各个部件指定抽象接口
- ConcreteBuilder：实现Builder的接口以构造和装配该产品的各个部件；定义并跟踪它所创建的表示；提供一个检索产品的接口
- Director：构造一个使用Builder接口的对象
- Product：表示被构造的复杂对象，ConcreteBuilder创建该产品的内部表示并定义它的装配过程；包含定义组成部件的类，包括将这些部件装配成最终产品的接口

### 协作

- 客户创建Directer对象，并用他想用的Builder对象进行配置
- 一但生成了产品部件，导向器就会通知生成器
- 生成器处理导向器的请求，将部件添加到产品中
- 客户从生成器检索产品

![avatar](/image/生成器协作图.png)

### 效果

1. 使客户可以改变一个产品的内部表示：
2. 它将构造代码和表示代码分开：
3. 可以对构造过程进行更精细的控制：

### 实现

我们在游戏里创建一个角色，需要确定该角色的各种信息：

```c++
class Person {
public:
    Person(int f, color s, color h, tool w)
        :faceType(f), skinColor(s), hairColor(h), weapon(w) {};
private:
    int faceType;
    color skinColor;
    color hairColor;
    tool weapon;
};
```

随着角色信息变多，构造函数就会需要非常多的参数：`Person(10, yellow, red, gun, ......)`，这种时候可以先声明一个生成器类：

```c++
class PersonBuilder {
public:
    virtual void BuildPerson();
    virtual void BuildFace() {}
    virtual void BuildSkin() {}
    virtual void BuildHair() {}
    virtual void BuildWeapon() {}
    
    virtual Person* GetPerson() const { return 0; }
protected:
    PersonBuilder();
};
```

接着定义一个指导Builder创建Person类的Director类：

```c++
class PersonDirector {
public:
    PersonDirector();
    
    void Construct(PersonBuilder& builder)；
};
```

```c++
void PersonDiector::Construct(PersonBuilder &builder) {
    builder.BuildPerson();
    builder.BuildFace();
    builder.BuildSkin();
    builder.BuildHair();
    builder.BuildWeapon();
}
```

实际应用中很多时候Product自己就是Director，可以指导Builder对自己进行构造。同时要注意Builder也不直接参与构造Product，而是交给它的子类，不同的子类构造不同的Product：

```c++
class StandardPersonBuilder : public PersonBuilder {
public:
    StandardPersonBuilder();
    
    virtual void BuildPerson() { this.curPerson = new Person; }
    virtual void BuildFace() { this.curPerson->faceType = 10; }
    virtual void BuildSkin() { this.curPerson->skinColor = yellow; }
    virtual void BuildHair() { this.curPerson->hairColor = red; }
    virtual void BuildWeapon() { this.curPerson->weapon = gun; }
    
    virtual Person* GetPerson() const { return curPerson; }
private:
    Person* curPerson;
};
```

客户只需要创建一个`StandardPersonBuilder`作为函数`Construct()`的参数，就可以构造一个标准角色：

```c++
PersonDirector director;
StandardPersonBuilder builder;
director.Construct(&builder);
Person* man = builder.GetPerson();
```

### 技巧

1. 装配和构造接口：Builder类接口必须足够普遍，以便为各种类型的具体生成器构造产品
2. 产品没有抽象类：通常由具体生成器生成的产品，其表示相差非常大，给不同的产品以公共父类没有太大意义
3. 再Builder中缺省的方法为空：C++中，故意不把生成方法声明为纯虚成员函数，而是定义为空方法，让客户只重定义他们感兴趣的操作
