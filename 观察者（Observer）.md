# 观察者（Observer）

### 意图

定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖它的对象都得到通知并被自动更新

### 别名

依赖（dependent）、发布—订阅（publish-subscribe）

### 适用性

- 一个抽象的模型有两个方面，其中一个方面依赖于另一方面。将这二者封装在独立的对象中，以使它们可以独自地改变和复用
- 对一个对象的改变需要同时改变其他对象，而不知道具体有多少对象待改变
- 一个对象必须通知其他对象，而又不能假定其他对象是谁

### 结构

![avatar](image/观察者结构图.png)

### 参与者

- Subject：
  - 目标知道它的观察者，可以有任意多个观察者观察同一个目标
  - 提供注册和删除观察者对象的接口
- Observer：为那些在目标发生改变时需要获得通知的对象定义一个更新接口
- ConcreteSubject：
  - 将有关状态存入各ConcreteObserver对象
  - 状态发生改变时，向观察者发出通知
- ConcreteObserver：
  - 维护一个指向ConcreteSubject对象的引用
  - 存储有关状态，这些状态应该与目标状态保持一致
  - 实现Observer更新接口

### 协作

- ConcreteSubject发生任何改变时，通知观察者
- ConcreteObserver收到通知后，向ConcreteSubject查询消息，以使自身和目标的状态保持一致

![avatar](image/观察者协作图.png)

### 效果

优点：

- 目标和观察者间的抽象耦合：目标不知道任何一个观察者属于哪个具体的类，这样目标和观察者之间的耦合是抽象的和最小的，它们可以处于一个系统中的不同抽象层次
- 支持广播通信：拥有时刻增加和删除观察者的自由，处理还是忽略一个通知取决于观察者

缺点：

- 意外的更新：因为观察者之间不知道彼此的存在，所以它可能不知道改变目标会导致什么结果，一个在目标上看似无害的操作可能会引起一系列对观察者以及依赖于这些观察者的对象的更新

### 实例

一个网站上有很多视频博主，如果你喜欢一个博主的视频可以成为他的粉丝，之后网站会及时通知你该播主投稿了新视频。这里需要用到观察者模式：

```c++
class Observer {
public:
    virtual ~Observer();
    virtual void Update(Subject* theChangeSubject) = 0;
protected:
    Observer();
};
```

```c++
class Subject {
public:
    virtual ~Subject();
    
    virtual void Attach(Observer*);
    virtual void Detach(Observer*);
    virtual void Notify();
protected:
    Subject();
private:
    List<Observer*>* _observers;
};

void Subject::Attach(Observer* o) {
    _observers->Append(o);
}

void Subject::Detach(Observer* o) {
    _observers->Remove(o);
}

void Subject::Notify() {
    ListIterator<Observer*> iter(_observer);
    
    for (iter.First(); !iter.IsDone(); iter.Next()) {
        iter.CurrentItem()->Update(this);
    }
}
```

我们把博主作为Subject，粉丝作为Observer。注意和传统观察者模式不同，粉丝自身也维护一个关注的博主列表：

```c++
class Blogger : public Subject {
public:
    Blogger(const char* n) : _name(n) {}
    
    void Upload(Video*);
    const char* GetName() const;
    Iterator<Video*> GetVideos() const;
    
    // 其他博主操作...
    
private:
    const char* _name;
    List<Video*>* _videos;
};

void Blogger::Upload(Video* v) {
    // 上传视频...
    _videos->Append(v);
    Notify();
}

const char* Blogger::GetName() const {
    return _name;
}

Iterator<Video*> GetVideos() const {
    return Iterator<Video*>(_videos);
}
```

```c++
class Fan : public Observer {
public:
    Fan(const char* n) : _name(n) {}
    
    virtual void Subscribe(Blogger*);
    virtual void Unsubscribe(Blogger*);
    virtual void Update(Subject*);
    
    // 其他粉丝操作...
    
private:
    const char* _name;
    List<Blogger*>* _bloggers;
};

void Fan::Subscribe(Blogger* b) {
    _bloggers->Append(b);
    b->Attach(this);
    printf("已成功关注%s\n", b->GetName());
}

void Fan::Unsubscribe(Blogger* b) {
    _bloggers->Remove(b);
    b->Detach(this);
    printf("已取消关注%s\n", b->GetName());
}

void Fan::Update(Subject* theChangeSubject) {
    Iterator<Video*> iter = theChangeSubject->GetVideos();
    Video* newVideo = iter.Last()->CurrentItem();
    printf("你关注的博主：%s 刚刚发布了新视频：%s\n", theChangeSubject->GetName(), newVideo->Name());
}
```

现在就可以及时通知所有粉丝他们关注博主的更新情况：

```c++
Blogger* sun = new Blogger("Sun");
Video* v = new Video("Come here!");

Fan* sunKnight = Fan("SunKnight");
f.Subscribe(sun);

b.Upload(v);
```

上面代码运行的结果：

```c++
已成功关注Sun
你关注的博主：Sun 刚刚发布了新视频：Come here！
```

