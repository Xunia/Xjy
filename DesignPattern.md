
## 分类：

### 创建型模式
* 单例模式(Singleton) ☆☆☆
* 工厂方法模式(Factory Method) ☆☆
* 抽象工厂模式(Abstract Factory) ☆☆
* 建造者模式(Builder) ☆
* 原型模式(Prototype)
### 结构型模式
* 适配器模式(Adapter) ☆☆☆
* 装饰模式(Decorator) ☆☆☆
* 外观模式(Facade) ☆☆
* 组合模式(Composite) ☆
* 代理模式(Proxy) ☆
* 桥接模式(Bridge)
* 享元模式(Flyweight)
### 行为型模式
* 策略模式(Strategy) ☆☆☆
* 模板方法模式(Template Method) ☆☆☆
* 观察者模式(Observer) ☆☆
* 命令模式(Command) ☆☆
* 状态模式(State) ☆
* 职责链模式(Chain of Responsibility)
* 解释器模式(Interpreter)
* 迭代器模式(Iterator)
* 中介者模式(Mediator)
* 备忘录模式(Memento)
* 访问者模式(Visitor)


## 实例
### 1. 单例模式(Singleton)
饿汉式: 
```
class Singleton {
    private static Singleton instance = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return instance;
    }
}
```

懒汉式：
```
class Singleton {

    private static volatile Singleton singleton = null;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

### 2. 工厂方法模式(Factory Method)

分为三种：
1. 普通工厂模式，就是建立一个工厂类，对实现了同一接口的一些类进行实例的创建。 
2. 多个工厂方法模式，是对普通工厂方法模式的改进，在普通工厂方法模式中，如果传递的字符串出错，则不能正确创建对象，而多个工厂方法模式是提供多个工厂方法，分别创建对象。 
3. 静态工厂方法模式，将上面的多个工厂方法模式里的方法置为静态的，不需要创建实例，直接调用即可。


普通工厂模式：
```
interface Sender {
    void Send();
}

class MailSender implements Sender {

    @Override
    public void Send() {
        System.out.println("This is mail sender...");
    }
}

class SmsSender implements Sender {

    @Override
    public void Send() {
        System.out.println("This is sms sender...");
    }
}

public class FactoryPattern {
    public static void main(String[] args) {
        Sender sender = produce("mail");
        sender.Send();
    }
    public static Sender produce(String str) {
        if ("mail".equals(str)) {
            return new MailSender();
        } else if ("sms".equals(str)) {
            return new SmsSender();
        } else {
            System.out.println("输入错误...");
            return null;
        }
    }
}
```

多个工厂方法模式：
```
interface Sender {
    void Send();
}

class MailSender implements Sender {

    @Override
    public void Send() {
        System.out.println("This is mail sender...");
    }
}

class SmsSender implements Sender {

    @Override
    public void Send() {
        System.out.println("This is sms sender...");
    }
}

class SendFactory {
    public static Sender produceMail() {
        return new MailSender();
    }

    public static Sender produceSms() {
        return new SmsSender();
    }
}

public class FactoryPattern {
    public static void main(String[] args) {
        Sender sender = SendFactory.produceMail();
        sender.Send();
    }
}
```

静态工厂方法模式：

```
interface Sender {
    void Send();
}

class MailSender implements Sender {

    @Override
    public void Send() {
        System.out.println("This is mail sender...");
    }
}

class SmsSender implements Sender {

    @Override
    public void Send() {
        System.out.println("This is sms sender...");
    }
}

class SendFactory {
    public static Sender produceMail() {
        return new MailSender();
    }

    public static Sender produceSms() {
        return new SmsSender();
    }
}

public class FactoryPattern {
    public static void main(String[] args) {
        Sender sender = SendFactory.produceMail();
        sender.Send();
    }
}
```

### 3. 抽象工厂模式

```
interface Provider {
    Sender produce();
}

interface Sender {
    void Send();
}

class MailSender implements Sender {

    public void Send() {
        System.out.println("This is mail sender...");
    }
}

class SmsSender implements Sender {

    public void Send() {
        System.out.println("This is sms sender...");
    }
}

class SendMailFactory implements Provider {

    public Sender produce() {
        return new MailSender();
    }
}

class SendSmsFactory implements Provider {

    public Sender produce() {
        return new SmsSender();
    }
}


public class FactoryPattern {
    public static void main(String[] args) {
        Provider provider = new SendMailFactory();
        Sender sender = provider.produce();
        sender.Send();
    }
}
```
### 4. 适配器模式(Adapter)
主要分三类：类的适配器模式、对象的适配器模式、接口的适配器模式。

类的适配器模式:
```
class Source {
    public void method1() {
        System.out.println("This is original method...");
    }
}

interface Targetable {

    /**
     * 与原类中的方法相同
     */
    public void method1();

    /**
     * 新类的方法
     */
    public void method2();
}

class Adapter extends Source implements Targetable {

    @Override
    public void method2() {
        System.out.println("This is the targetable method...");
    }
}

public class AdapterPattern {
    public static void main(String[] args) {
        Targetable targetable = new Adapter();
        targetable.method1();
        targetable.method2();
    }
}
```

对象的适配器模式：
```
class Source {
    public void method1() {
        System.out.println("This is original method...");
    }
}

interface Targetable {

    /**
     * 与原类中的方法相同
     */
    public void method1();

    /**
     * 新类的方法
     */
    public void method2();
}

class Wrapper implements Targetable {

    private Source source;

    public Wrapper(Source source) {
        super();
        this.source = source;
    }

    @Override
    public void method1() {
        source.method1();
    }

    @Override
    public void method2() {
        System.out.println("This is the targetable method...");
    }
}

public class AdapterPattern {
    public static void main(String[] args) {
        Source source = new Source();
        Targetable targetable = new Wrapper(source);
        targetable.method1();
        targetable.method2();
    }
}
```

接口的适配器模式:
```
/**
 * 定义端口接口，提供通信服务
 */
interface Port {
    /**
     * 远程SSH端口为22
     */
    void SSH();

    /**
     * 网络端口为80
     */
    void NET();

    /**
     * Tomcat容器端口为8080
     */
    void Tomcat();

    /**
     * MySQL数据库端口为3306
     */
    void MySQL();
}

/**
 * 定义抽象类实现端口接口，但是什么事情都不做
 */
abstract class Wrapper implements Port {
    @Override
    public void SSH() {

    }

    @Override
    public void NET() {

    }

    @Override
    public void Tomcat() {

    }

    @Override
    public void MySQL() {

    }
}

/**
 * 提供聊天服务
 * 需要网络功能
 */
class Chat extends Wrapper {
    @Override
    public void NET() {
        System.out.println("Hello World...");
    }
}

/**
 * 网站服务器
 * 需要Tomcat容器，Mysql数据库，网络服务，远程服务
 */
class Server extends Wrapper {
    @Override
    public void SSH() {
        System.out.println("Connect success...");
    }

    @Override
    public void NET() {
        System.out.println("WWW...");
    }

    @Override
    public void Tomcat() {
        System.out.println("Tomcat is running...");
    }

    @Override
    public void MySQL() {
        System.out.println("MySQL is running...");
    }
}

public class AdapterPattern {

    private static Port chatPort = new Chat();
    private static Port serverPort = new Server();

    public static void main(String[] args) {
        // 聊天服务
        chatPort.NET();

        // 服务器
        serverPort.SSH();
        serverPort.NET();
        serverPort.Tomcat();
        serverPort.MySQL();
    }
}
```

### 5. 装饰模式(Decorator)

```
interface Shape {
    void draw();
}

/**
 * 实现接口的实体类
 */
class Rectangle implements Shape {

    @Override
    public void draw() {
        System.out.println("Shape: Rectangle...");
    }
}

class Circle implements Shape {

    @Override
    public void draw() {
        System.out.println("Shape: Circle...");
    }
}

/**
 * 创建实现了 Shape 接口的抽象装饰类。
 */
abstract class ShapeDecorator implements Shape {
    protected Shape decoratedShape;

    public ShapeDecorator(Shape decoratedShape) {
        this.decoratedShape = decoratedShape;
    }

    @Override
    public void draw() {
        decoratedShape.draw();
    }
}

/**
 *  创建扩展自 ShapeDecorator 类的实体装饰类。
 */
class RedShapeDecorator extends ShapeDecorator {

    public RedShapeDecorator(Shape decoratedShape) {
        super(decoratedShape);
    }

    @Override
    public void draw() {
        decoratedShape.draw();
        setRedBorder(decoratedShape);
    }

    private void setRedBorder(Shape decoratedShape) {
        System.out.println("Border Color: Red");
    }
}

/**
 * 使用 RedShapeDecorator 来装饰 Shape 对象。
 */
public class DecoratorPattern {
    public static void main(String[] args) {
        Shape circle = new Circle();
        Shape redCircle = new RedShapeDecorator(new Circle());
        Shape redRectangle = new RedShapeDecorator(new Rectangle());
        System.out.println("Circle with normal border");
        circle.draw();

        System.out.println("\nCircle of red border");
        redCircle.draw();

        System.out.println("\nRectangle of red border");
        redRectangle.draw();
    }
}
```