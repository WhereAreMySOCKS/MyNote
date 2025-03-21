# 设计模式

## 单例模式
单例模式（Singleton Pattern）是一种创建型设计模式，确保一个类只有一个实例，并提供一个全局访问点。

### 应用场景
1. 需要频繁实例化然后销毁的对象
2. 创建对象时耗时过多或耗资源过多，但又经常用到的对象
3. 有状态的工具类对象
4. 频繁访问数据库或文件的对象

### 实现方式
#### 1. 饿汉式
```java
public class Singleton {
    // 在类加载时就创建实例
    private static final Singleton instance = new Singleton();
    
    // 私有构造方法，防止外部创建实例
    private Singleton() {}
    
    // 提供全局访问点
    public static Singleton getInstance() {
        return instance;
    }
}
```
特点：线程安全，但不管是否用到都会创建实例，可能造成资源浪费。

#### 2. 懒汉式（线程不安全）
```java
public class Singleton {
    // 声明但不创建实例
    private static Singleton instance;
    
    private Singleton() {}
    
    // 在第一次调用时创建实例
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
特点：延迟加载，但在多线程环境下可能创建多个实例。

#### 3. 懒汉式（线程安全）
```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {}
    
    // 使用synchronized关键字保证线程安全
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
特点：线程安全，但同步方法效率低。

#### 4. 双重检查锁
```java
public class Singleton {
    // 使用volatile关键字保证可见性和有序性
    private static volatile Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
特点：线程安全，且大部分情况下无需同步，兼顾效率与安全性。

#### 5. 静态内部类
```java
public class Singleton {
    private Singleton() {}
    
    // 静态内部类在外部类加载时不会被加载
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
特点：利用类加载机制保证线程安全，实现延迟加载，是一种推荐的实现方式。

#### 6. 枚举
```java
public enum Singleton {
    INSTANCE;
    
    // 添加单例需要的方法
    public void doSomething() {
        // 业务逻辑
    }
}
```
特点：最简洁的实现，自动支持序列化，绝对防止多次实例化，是实现单例的最佳方式。

## 工厂模式
工厂模式（Factory Pattern）是一种创建型设计模式，提供了一种创建对象的最佳方式。在工厂模式中，创建对象时不会对客户端暴露创建逻辑，而是通过使用一个共同的接口来指向新创建的对象。

### 工厂模式的三种类型

#### 1. 简单工厂模式
简单工厂模式不是一个标准的设计模式，更像是一种编程习惯。它通过一个工厂类根据传入的参数决定创建出哪一种产品类的实例。

```java
// 产品接口
interface Product {
    void operation();
}

// 具体产品A
class ConcreteProductA implements Product {
    @Override
    public void operation() {
        System.out.println("ConcreteProductA operation");
    }
}

// 具体产品B
class ConcreteProductB implements Product {
    @Override
    public void operation() {
        System.out.println("ConcreteProductB operation");
    }
}

// 简单工厂
class SimpleFactory {
    public static Product createProduct(String type) {
        if ("A".equals(type)) {
            return new ConcreteProductA();
        } else if ("B".equals(type)) {
            return new ConcreteProductB();
        }
        return null;
    }
}

// 客户端代码
public class Client {
    public static void main(String[] args) {
        Product productA = SimpleFactory.createProduct("A");
        productA.operation();
        
        Product productB = SimpleFactory.createProduct("B");
        productB.operation();
    }
}
```

优点：实现简单，客户端无需知道具体产品类名，只需知道产品对应的参数即可。
缺点：扩展新产品需要修改工厂类代码，违反开闭原则。

#### 2. 工厂方法模式
工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法让类的实例化推迟到子类中进行。

```java
// 产品接口
interface Product {
    void operation();
}

// 具体产品A
class ConcreteProductA implements Product {
    @Override
    public void operation() {
        System.out.println("ConcreteProductA operation");
    }
}

// 具体产品B
class ConcreteProductB implements Product {
    @Override
    public void operation() {
        System.out.println("ConcreteProductB operation");
    }
}

// 工厂接口
interface Factory {
    Product createProduct();
}

// 具体工厂A
class ConcreteFactoryA implements Factory {
    @Override
    public Product createProduct() {
        return new ConcreteProductA();
    }
}

// 具体工厂B
class ConcreteFactoryB implements Factory {
    @Override
    public Product createProduct() {
        return new ConcreteProductB();
    }
}

// 客户端代码
public class Client {
    public static void main(String[] args) {
        Factory factoryA = new ConcreteFactoryA();
        Product productA = factoryA.createProduct();
        productA.operation();
        
        Factory factoryB = new ConcreteFactoryB();
        Product productB = factoryB.createProduct();
        productB.operation();
    }
}
```

优点：符合开闭原则，新增产品只需添加具体产品类和对应的工厂类，不需修改原有代码。
缺点：类的数量增加，增加了系统的复杂度。

#### 3. 抽象工厂模式
抽象工厂模式提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

```java
// 产品A接口
interface ProductA {
    void operationA();
}

// 产品B接口
interface ProductB {
    void operationB();
}

// 具体产品A1
class ConcreteProductA1 implements ProductA {
    @Override
    public void operationA() {
        System.out.println("ConcreteProductA1 operationA");
    }
}

// 具体产品A2
class ConcreteProductA2 implements ProductA {
    @Override
    public void operationA() {
        System.out.println("ConcreteProductA2 operationA");
    }
}

// 具体产品B1
class ConcreteProductB1 implements ProductB {
    @Override
    public void operationB() {
        System.out.println("ConcreteProductB1 operationB");
    }
}

// 具体产品B2
class ConcreteProductB2 implements ProductB {
    @Override
    public void operationB() {
        System.out.println("ConcreteProductB2 operationB");
    }
}

// 抽象工厂
interface AbstractFactory {
    ProductA createProductA();
    ProductB createProductB();
}

// 具体工厂1
class ConcreteFactory1 implements AbstractFactory {
    @Override
    public ProductA createProductA() {
        return new ConcreteProductA1();
    }
    
    @Override
    public ProductB createProductB() {
        return new ConcreteProductB1();
    }
}

// 具体工厂2
class ConcreteFactory2 implements AbstractFactory {
    @Override
    public ProductA createProductA() {
        return new ConcreteProductA2();
    }
    
    @Override
    public ProductB createProductB() {
        return new ConcreteProductB2();
    }
}

// 客户端代码
public class Client {
    public static void main(String[] args) {
        AbstractFactory factory1 = new ConcreteFactory1();
        ProductA productA1 = factory1.createProductA();
        ProductB productB1 = factory1.createProductB();
        productA1.operationA();
        productB1.operationB();
        
        AbstractFactory factory2 = new ConcreteFactory2();
        ProductA productA2 = factory2.createProductA();
        ProductB productB2 = factory2.createProductB();
        productA2.operationA();
        productB2.operationB();
    }
}
```

优点：当一个产品族中的多个对象被设计成一起工作时，它能保证客户端始终只使用同一个产品族中的对象。
缺点：产品族扩展困难，如果要增加一个产品族中的产品，需要修改抽象工厂的接口和所有的具体工厂类，这违反了开闭原则。

## 动态代理
动态代理（Dynamic Proxy）是一种设计模式，它允许在运行时创建一个代理对象，用于代替真实对象的功能。代理对象可以在不改变原始代码的情况下，对方法调用进行拦截、增强或者修改。在动态代理中，代理对象不是事先编写好的类，而是在程序运行时根据接口或类的信息动态生成的。

### 动态代理的应用场景
1. 日志记录
2. 性能统计
3. 权限控制
4. 事务处理
5. 延迟加载
6. 远程调用

### JDK动态代理
JDK动态代理是Java自带的动态代理机制，它要求被代理的类必须实现接口。

```java
// 定义接口
interface Service {
    void lookConcert();
    void seeADoctor();
    void shopping();
}

// 实现接口的真实对象
class RealService implements Service {
    @Override
    public void lookConcert() {
        System.out.println("看演唱会");
    }

    @Override
    public void seeADoctor() {
        System.out.println("看医生");
    }

    @Override
    public void shopping() {
        System.out.println("去购物");
    }
}

// 实现InvocationHandler接口的代理处理器
public class HuangNiuHandle implements InvocationHandler {
    private Object proxyTarget;
    
    // 使用反射动态创建代理对象
    public Object getProxyInstance(Object target) {
        this.proxyTarget = target;
        return Proxy.newProxyInstance(
            proxyTarget.getClass().getClassLoader(), 
            proxyTarget.getClass().getInterfaces(), 
            this
        );
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object methodObject = null;
        if ("lookConcert".equals(method.getName()) || "seeADoctor".equals(method.getName())) {
            System.out.println("排队"); // 增强逻辑
            methodObject = method.invoke(proxyTarget, args); // 调用目标方法
        } else {
            // 不使用第一个proxy参数作为参数，否则会造成死循环
            methodObject = method.invoke(proxyTarget, args);
        }
        return methodObject;
    }
}

// 客户端代码
public class Client {
    public static void main(String[] args) {
        // 创建真实对象
        Service realService = new RealService();
        
        // 创建代理对象
        HuangNiuHandle handle = new HuangNiuHandle();
        Service proxyService = (Service) handle.getProxyInstance(realService);
        
        // 通过代理对象调用方法
        proxyService.lookConcert(); // 会先排队，再看演唱会
        proxyService.seeADoctor();  // 会先排队，再看医生
        proxyService.shopping();    // 直接去购物，没有增强
    }
}
```

### CGLIB动态代理
CGLIB（Code Generation Library）是一个强大的高性能代码生成库，它可以在运行时扩展Java类并实现接口。当被代理类没有实现接口时，可以使用CGLIB实现动态代理。

```java
// 需要导入CGLIB依赖
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

// 目标类，没有实现任何接口
class TargetObject {
    public void method1() {
        System.out.println("方法1");
    }
    
    public void method2() {
        System.out.println("方法2");
    }
}

// 实现MethodInterceptor接口的代理处理器
class CglibProxy implements MethodInterceptor {
    private Object target;
    
    public CglibProxy(Object target) {
        this.target = target;
    }
    
    // 创建代理对象
    public Object getProxyInstance() {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(target.getClass());
        enhancer.setCallback(this);
        return enhancer.create();
    }
    
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("开始代理");
        Object result = method.invoke(target, args);
        System.out.println("结束代理");
        return result;
    }
}

// 客户端代码
public class Client {
    public static void main(String[] args) {
        // 创建目标对象
        TargetObject target = new TargetObject();
        
        // 创建代理对象
        CglibProxy proxy = new CglibProxy(target);
        TargetObject proxyInstance = (TargetObject) proxy.getProxyInstance();
        
        // 通过代理对象调用方法
        proxyInstance.method1();
        proxyInstance.method2();
    }
}
```

### JDK动态代理与CGLIB动态代理的区别
1. **实现方式不同**：
   - JDK动态代理是通过接口实现的，要求被代理的类必须实现接口。
   - CGLIB动态代理是通过继承目标类生成代理子类，因此无法代理final类或final方法。

2. **性能差异**：
   - 在早期版本中，CGLIB的性能通常优于JDK动态代理。
   - 在现代JDK版本（如JDK 8及以上）中，两者性能差距已经不大，甚至在某些场景下JDK动态代理可能更快。

3. **应用场景**：
   - 如果目标类有实现接口，通常选择JDK动态代理。
   - 如果目标类没有实现接口，则必须使用CGLIB动态代理。

4. **SpringAOP中的应用**：
   - Spring AOP默认使用JDK动态代理实现。
   - 当被代理的类没有实现接口时，Spring会自动切换为CGLIB动态代理。 