# synchronized同步锁

## 概述
synchronized作为多线程关键字，是一种同步锁，它可以修饰以下几种对象：  
代码块：被修饰的代码块称为同步语句块，其作用的范围是大括号{ }里的代码，作用的对象是调用这个代码块的对象；但是由于synchronized时的对象属性区别（是否是static变量），其作用范围也是有区别的；  
方法：被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象  
静态方法：作用的范围是整个静态方法，作用的对象是这个类的所有对象  
类：作用的范围是synchronize后面括号里的部分，作用的对象是这个类的所有对象  
  
根据其作用的范围可分为对象错和类锁。  

----

## 具体使用种类
### 1. 修饰方法
```java
synchronized public void getValue() {
    System.out.println("getValue method thread name="
            + Thread.currentThread().getName() + " username=" + username
            + " password=" + password);
}
```
### 2. 修饰代码块
```java
public void serviceMethod() {
    try {
        synchronized (this) {
            System.out.println("begin time=" + System.currentTimeMillis());
            Thread.sleep(2000);
            System.out.println("end    end=" + System.currentTimeMillis());
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```
### 3. 修饰类
```java
public static void printA() {
        synchronized (Service.class) {
            try {
                System.out.println("线程名称为：" + Thread.currentThread().getName()
                        + "在" + System.currentTimeMillis() + "进入printA");
                Thread.sleep(3000);
                System.out.println("线程名称为：" + Thread.currentThread().getName()
                        + "在" + System.currentTimeMillis() + "离开printA");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
```
----

## 用法实例
synchronize修饰非静态方法、同步代码块的synchronize(this)和synchronize(非this对象/非static型对象)的用法锁的是对象，线程想要执行对应的同步代码，需要获得`对象锁`。  
synchronize修饰静态方法以及同步代码块的synchronize(类.class)用法锁是类，synchronize(static object)同样锁的是类，线程想要执行对应的同步代码，需要获得`类锁`。  

### 1. 线程不安全实例
```java
public class Run {

    public static void main(String[] args) {
        HasSelfPrivateNum numRef = new HasSelfPrivateNum();
        ThreadA athread = new ThreadA(numRef);
        athread.start();
        ThreadB bthread = new ThreadB(numRef);
        bthread.start();
    }
}

class HasSelfPrivateNum {
    private int num = 0;
    public void addI(String username) {
        try {
            if (username.equals("a")) {
                num = 100;
                System.out.println("a set over!");
                Thread.sleep(2000);
            } else {
                num = 200;
                System.out.println("b set over!");
            }
            System.out.println(username + " num=" + num);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

class ThreadA extends Thread {

    private HasSelfPrivateNum numRef;

    public ThreadA(HasSelfPrivateNum numRef) {
        super();
        this.numRef = numRef;
    }

    @Override
    public void run() {
        super.run();
        numRef.addI("a");
    }
}

 class ThreadB extends Thread {

    private HasSelfPrivateNum numRef;

    public ThreadB(HasSelfPrivateNum numRef) {
        super();
        this.numRef = numRef;
    }

    @Override
    public void run() {
        super.run();
        numRef.addI("b");
    }

}
```
输出结果：  
```
a set over!
b set over!
b num=200
a num=200
```
修改HasSelfPrivateNum如下，方法用synchronize修饰如下：  

```java
class HasSelfPrivateNum {

    private int num = 0;

    public synchronized void addI(String username) {
        try {
            if (username.equals("a")) {
                num = 100;
                System.out.println("a set over!");
                Thread.sleep(2000);
            } else {
                num = 200;
                System.out.println("b set over!");
            }
            System.out.println(username + " num=" + num);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}
```
输出结果：  
```
b set over!
b num=200
a set over!
a num=100
```
sleep方法不会放弃对象锁，需要等到时间结束，释放锁，下一个进程才能获取到该对象锁  
实验结论：两个线程访问同一个对象中的同步方法是一定是线程安全的。本实现由于是同步访问，所以先打印出a，然后打印出b。  
这里线程获取的是HasSelfPrivateNum的对象实例的锁——`对象锁`。  

### 2. 多个对象多个锁
```java
public class Run {

    public static void main(String[] args) {

        HasSelfPrivateNum numRef1 = new HasSelfPrivateNum();
        HasSelfPrivateNum numRef2 = new HasSelfPrivateNum();

        ThreadA athread = new ThreadA(numRef1);
        athread.start();

        ThreadB bthread = new ThreadB(numRef2);
        bthread.start();
    }
}
```
输出结果：  
```
a set over!
b set over!
b num=200
a num=200
```
这里是非同步的，由于是多个线程调用多个对象，不涉及线程间的同步。因为线程athread获得是numRef1的对象锁，而bthread线程获取的是numRef2的对象锁，他们并没有在获取锁上有竞争关系，因此，出现非同步的结果。  

### 3. 同步块synchronize(this)
```java
public class Run {
    public static void main(String[] args) {
        ObjectService service = new ObjectService();

        ThreadA a = new ThreadA(service);
        a.setName("a");
        a.start();

        ThreadB b = new ThreadB(service);
        b.setName("b");
        b.start();
    }
}

class ObjectService {

    public void serviceMethod() {
        try {
            synchronized (this) {
                System.out.println("begin time=" + System.currentTimeMillis());
                Thread.sleep(2000);
                System.out.println("end    end=" + System.currentTimeMillis());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}


class ThreadA extends Thread {

    private ObjectService service;

    public ThreadA(ObjectService service) {
        super();
        this.service = service;
    }

    @Override
    public void run() {
        super.run();
        service.serviceMethod();
    }

}

class ThreadB extends Thread {
    private ObjectService service;
    public ThreadB(ObjectService service) {
        super();
        this.service = service;
    }

    @Override
    public void run() {
        super.run();
        service.serviceMethod();
    }
}
```
输出结果：  
```
begin time=1466148260341
end    end=1466148262342
begin time=1466148262342
end    end=1466148264378
```
同步，线程获取的是同步块synchronized (this)括号（）里面的对象实例的对象锁，这里就是ObjectService实例对象的对象锁了。  

### 4. synchronize(非this对象)
```java
public class Run {
    public static void main(String[] args) {

        Service service = new Service("xiaobaoge");

        ThreadA a = new ThreadA(service);
        a.setName("A");
        a.start();

        ThreadB b = new ThreadB(service);
        b.setName("B");
        b.start();
    }
}

class Service {
    String anyString = new String();
    public Service(String anyString){
        this.anyString = anyString;
    }

    public void setUsernamePassword(String username, String password) {
        try {
            synchronized (anyString) {
                System.out.println("线程名称为：" + Thread.currentThread().getName()
                        + "在" + System.currentTimeMillis() + "进入同步块");
                Thread.sleep(3000);
                System.out.println("线程名称为：" + Thread.currentThread().getName()
                        + "在" + System.currentTimeMillis() + "离开同步块");
            }
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}

class ThreadA extends Thread {
    private Service service;
    public ThreadA(Service service) {
        super();
        this.service = service;
    }

    @Override
    public void run() {
        service.setUsernamePassword("a", "aa");
    }
}


class ThreadB extends Thread {
    private Service service;
    public ThreadB(Service service) {
        super();
        this.service = service;
    }

    @Override
    public void run() {
        service.setUsernamePassword("b", "bb");
    }
}
```
这里线程争夺的是anyString的对象锁，两个线程有竞争同一对象锁的关系，出现同步。 

### 5. synchronize(static object)
待补充  

### 6. 静态synchronize同步方法
```java
public class Run {

    public static void main(String[] args) {

        ThreadA a = new ThreadA();
        a.setName("A");
        a.start();

        ThreadB b = new ThreadB();
        b.setName("B");
        b.start();

    }

}

class Service {

    synchronized public static void printA() {
        try {
            System.out.println("线程名称为：" + Thread.currentThread().getName()
                    + "在" + System.currentTimeMillis() + "进入printA");
            Thread.sleep(3000);
            System.out.println("线程名称为：" + Thread.currentThread().getName()
                    + "在" + System.currentTimeMillis() + "离开printA");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    synchronized public static void printB() {
        System.out.println("线程名称为：" + Thread.currentThread().getName() + "在"
                + System.currentTimeMillis() + "进入printB");
        System.out.println("线程名称为：" + Thread.currentThread().getName() + "在"
                + System.currentTimeMillis() + "离开printB");
    }

}


class ThreadA extends Thread {
    @Override
    public void run() {
        Service.printA();
    }

}


class ThreadB extends Thread {
    @Override
    public void run() {
        Service.printB();
    }
}
```
输出结果：  
```
线程名称为：A在1466149372909进入printA
线程名称为：A在1466149375920离开printA
线程名称为：B在1466149375920进入printB
线程名称为：B在1466149375920离开printB
```
两个线程在争夺同一个类锁，因此同步。  
### 7. synchronize(class)
```java
class Service {

    public static void printA() {
        synchronized (Service.class) {
            try {
                System.out.println("线程名称为：" + Thread.currentThread().getName()
                        + "在" + System.currentTimeMillis() + "进入printA");
                Thread.sleep(3000);
                System.out.println("线程名称为：" + Thread.currentThread().getName()
                        + "在" + System.currentTimeMillis() + "离开printA");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

    public static void printB() {
        synchronized (Service.class) {
            System.out.println("线程名称为：" + Thread.currentThread().getName()
                    + "在" + System.currentTimeMillis() + "进入printB");
            System.out.println("线程名称为：" + Thread.currentThread().getName()
                    + "在" + System.currentTimeMillis() + "离开printB");
        }
    }
}
```
输出结果：  
```
线程名称为：A在1466149372909进入printA
线程名称为：A在1466149375920离开printA
线程名称为：B在1466149375920进入printB
线程名称为：B在1466149375920离开printB
```
两个线程依旧在争夺同一个类锁，因此同步。  

----
## 结论
1. synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法、一个类、或者某个静态变量，则它取得的锁是对类，该类所有的对象同一把锁。   
2. 无论是修饰静态方法还是锁定某个对象,都是 类锁.一个class其中的静态方法和静态变量在内存中只会加载和初始化一份，所以，一旦一个静态的方法被申明为synchronized，此类的所有的实例化对象在调用该方法时，共用同一把锁，称之为类锁。

## 备注
1. synchronized关键字不能继承。也就是说子类重写了父类中用synchronized修饰的方法，子类的方法仍然不是同步的。
2. 定义接口方法时，不能使用synchronized关键字。
3. 构造方法不能使用synchronized关键字，但是可以使用synchronized代码块。