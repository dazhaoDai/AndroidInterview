 代理模式是常用的Java设计模式，它的特征是代理类与委托类有同样的接口，一个代理类的对象与一个委托类的对象关联，**代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务**，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。
  
>简单举个例子，例如你现在正在看电视，但是老师布置了作业，你又想看电视又要完成作业，你灵机一动，让哥哥帮你写，你开心的看完电视，作业也写完了，这种让别人代替你完成的某种任务的行为就是代理模式

按照代理类的创建时期，代理类可分为两种。

####  静态代理：
由程序员创建或由特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了。实现类和代理类实现同一个接口，将实现类对象传递给代理类，代理类的实现方法实际是由实现类完成操作的。

```
public interface Subject {
    void visit();
}

//执行者，代理者
public class RealSubject implements Subject {

    private String name = "proxy";
    @Override
    public void visit() {
        System.out.println(name);
    }
}

//委托者，被代理者
public class ProxySubject implements Subject{
    private Subject subject;
    public ProxySubject(Subject subject) {
        this.subject = subject;
    }

    @Override
    public void visit() {
        subject.visit();
    }
}

//执行
public class Client {
    public static void main(String[] args) {
    	//真正执行的是代理者
        ProxySubject subject = new ProxySubject(new RealSubject());
        subject.visit();
    }
}
```

#### 动态代理
动态代理有别于静态代理，是根据代理的对象，动态创建代理类。这样，就可以避免静态代理中代理类接口过多的问题。动态代理是实现方式，是通过反射来实现的，借助Java自带的java.lang.reflect.Proxy,通过固定的规则生成。
其步骤如下：
- 编写一个委托类的接口，即静态代理的（Iuser接口）
- 实现一个真正的委托类，即静态代理的（UserImpl类）
- 创建一个动态代理类，实现InvocationHandler接口，并重写该invoke方法
在测试类中，生成动态代理的对象。

第一二步骤，和静态代理一样，不过说了。第三步，代码如下：
动态代理Demo
```
public interface Iuser {
 　　void eat(String s);
 }
 //委托类，执行者
public class UserImpl implements Iuser {
　　@Override
　　public void eat(String s) {
　　　　System.out.println("我要吃"+s);
　　}
}

public class DynamicProxy implements InvocationHandler {
　　private Object object;//用于接收具体实现类的实例对象
　　//使用带参数的构造器来传递具体实现类的对象
　　public DynamicProxy(Object obj){
　　　　this.object = obj;
　　}
　　@Override
　　public Object invoke(Object proxy, Method method, Object[] args)throws Throwable {
　　　　Object result = method.invoke(object, args);
　　　　return result;
　　}
}

public class ProxyTest {
　　public static void main(String[] args) {
　　　　Iuser user = new UserImpl();
　　　　InvocationHandler h = new DynamicProxy(user);
　　　　Iuser proxy = (Iuser) Proxy.newProxyInstance(user..getClass().getClassLoader(), new Class[]{Iuser.class}, h);
　　　　proxy.eat("苹果");
　　}
}

```

创建动态代理的对象，需要借助Proxy.newProxyInstance。该方法的三个参数分别是：

- ClassLoader loader表示当前使用到的appClassloader。
- Class<?>[] interfaces表示目标对象实现的一组接口。
- InvocationHandler h表示当前的InvocationHandler实现实例对象。
简单介绍了 静态代理和动态代理的区别