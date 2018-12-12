---
layout: post
title: java序列化与反序列化
tags: [jdk]
---
目录：
- [对象序列化](#对象序列化)
- [属性序列化的情况](#属性序列化的情况)
- [涉及到父类的情况](#涉及到父类的情况)
- [serialVersionUID](#serialversionuid)
- [transient关键字](#transient关键字)

最近将一个对象写入redis的时候遇到序列化与反序列化的问题，就专门研究了一下java中的序列化与反序列化。

## 对象序列化

在java中，如果想将一个对象序列化，那么被序列化的类需要实现Serializable接口，如果不实现会怎么样呢？直接上代码。
```java

/**
 * Created by ruanwenjun on 2018/12/11
 */
public class Dog {
    private String name;
    private int age;

    public Dog(String name, int age) {
        this.name = name;
        this.age = age;
    }

```

```java

/**
 * Created by ruanwenjun on 2018/12/11
 */
public class SerializableTest {

    public static void main(String[] args) {
        try (ObjectOutputStream objectOutputStream =
                     new ObjectOutputStream(new FileOutputStream("/Users/ruanwenjun/mydog"));
             ObjectInputStream objectInputStream =
                     new ObjectInputStream(new FileInputStream("/Users/ruanwenjun/mydog"))) {
            Dog dog = new Dog("tom", 1);
            objectOutputStream.writeObject(dog);
            dog = (Dog) objectInputStream.readObject();
            System.out.println(dog);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }

    }
}
```

运行main方法，控制台会抛异常
```java
java.io.NotSerializableException: com.wenjun.Dog
	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1184)
	at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:348)
	at com.wenjun.SerializableTest.main(SerializableTest.java:17)
```
查看ObjectOutputStream的writeObject方法声明
```java
@throws  NotSerializableException Some object to be serialized does not implement the java.io.Serializable interface.
```
表明如果一个类没有实现java.io.Serializable接口，那么就会抛NotSerializableException异常（至于为什么会这样设计 ？）。反正一个类要想能够序列化，必须实现序列化接口。

将上面的Dog类加上序列化接口即可执行成功（代码省略）。

## 属性序列化的情况

那么只要实现了Serializable接口就一定可以序列化成功吗？答案是否？
比如，改写上面的Dog类
```java


/**
 * Created by ruanwenjun on 2018/12/11
 */
public class Dog implements Serializable {
    private String name;
    private int age;
    private Host host;

    public Dog(String name, int age, Host host) {
        this.name = name;
        this.age = age;
        this.host = host;
    }

    @Override
    public String toString() {
        return "Dog{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", host=" + host +
                '}';
    }
}


/**
 * Created by ruanwenjun on 2018/12/11
 *
 */
public class Host {
    private String name;

    @Override
    public String toString() {
        return "Host{" +
                "name='" + name + '\'' +
                '}';
    }
}




/**
 * Created by ruanwenjun on 2018/12/11
 */
public class SerializableTest {

    public static void main(String[] args) {
        try (ObjectOutputStream objectOutputStream =
                     new ObjectOutputStream(new FileOutputStream("/Users/ruanwenjun/mydog"));
             ObjectInputStream objectInputStream =
                     new ObjectInputStream(new FileInputStream("/Users/ruanwenjun/mydog"))) {
            Dog dog = new Dog("tom", 1, new Host("tom father"));
            objectOutputStream.writeObject(dog);
            dog = (Dog) objectInputStream.readObject();
            System.out.println(dog);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }

    }
}

```
再次执行main函数，发现抛异常
```java
java.io.NotSerializableException: com.wenjun.Host
```
表明，一个类要想实现序列化，除了他自己要实现Serializable，他的所有属性都必须实现Serializable接口，将Host实现Serializable之后程序运行成功。

## 涉及到父类的情况

那么所有属性都实现Serializable接口就一定能序列化反序列化成功吗？也不一定
```java

/**
 * Created by ruanwenjun on 2018/12/11
 */
public class BaseDog {

    protected String hair;

    public BaseDog(String hair) {
        this.hair = hair;
    }
}


/**
 * Created by ruanwenjun on 2018/12/11
 */
public class Dog extends BaseDog implements Serializable {
    private String name;
    private int age;
    private Host host;

    public Dog(String hair, String name, int age, Host host) {
        super(hair);
        this.name = name;
        this.age = age;
        this.host = host;
    }

    @Override
    public String toString() {
        return "Dog{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", host=" + host +
                ", hair='" + hair + '\'' +
                '}';
    }
}



/**
 * Created by ruanwenjun on 2018/12/11
 */
public class SerializableTest {

    public static void main(String[] args) {
        try (ObjectOutputStream objectOutputStream =
                     new ObjectOutputStream(new FileOutputStream("/Users/ruanwenjun/mydog"));
             ObjectInputStream objectInputStream =
                     new ObjectInputStream(new FileInputStream("/Users/ruanwenjun/mydog"))) {
            Dog dog = new Dog("red", "tom", 1, new Host("tom father"));
            objectOutputStream.writeObject(dog);
            dog = (Dog) objectInputStream.readObject();
            System.out.println(dog);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }

    }
}

java.io.InvalidClassException: com.wenjun.Dog; no valid constructor
	at java.io.ObjectStreamClass$ExceptionInfo.newInvalidClassException(ObjectStreamClass.java:169)
	at java.io.ObjectStreamClass.checkDeserialize(ObjectStreamClass.java:874)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2043)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1573)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:431)
	at com.wenjun.SerializableTest.main(SerializableTest.java:18)


```

这个时候执行main函数，控制台会抛异常java.io.InvalidClassException: com.wenjun.Dog; no valid constructor , 这是说Dog没有有效的构造器,
Dog里面只有一个带参数的构造器，解决办法是加一个无参构造器，但是直接加会编译不通过，因为Dog的父类没有无参构造器。而且这个时候报错是在反序列化的时候。改写Dog如下
```java

/**
 * Created by ruanwenjun on 2018/12/11
 */
public class Dog extends BaseDog implements Serializable {
    private String name;
    private int age;
    private Host host;


    public Dog(){
    }

    public Dog(String hair, String name, int age, Host host) {
        super(hair);
        this.name = name;
        this.age = age;
        this.host = host;
    }

    @Override
    public String toString() {
        return "Dog{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", host=" + host +
                ", hair='" + hair + '\'' +
                '}';
    }
}

/**
 * Created by ruanwenjun on 2018/12/11
 */
public class BaseDog  {

    protected String hair;

    public BaseDog(String hair) {
        this.hair = hair;
    }

    public BaseDog(){}
}


/**
 * Created by ruanwenjun on 2018/12/11
 */
public class SerializableTest {

    public static void main(String[] args) {
        try (ObjectOutputStream objectOutputStream =
                     new ObjectOutputStream(new FileOutputStream("/Users/ruanwenjun/mydog"));
             ObjectInputStream objectInputStream =
                     new ObjectInputStream(new FileInputStream("/Users/ruanwenjun/mydog"))) {
            Dog dog = new Dog("red", "tom", 1, new Host("tom father"));
            objectOutputStream.writeObject(dog);
            dog = (Dog) objectInputStream.readObject();
            System.out.println(dog);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }

    }
}

Dog{name='tom', age=1, host=Host{name='tom father'}, hair='null'}

```
再次执行main方法，返回正常结果。这说明一个类要想反序列化,其父类必须有无参构造器,但是这个反序列化的输出结果并不正确，他的hair为null，表明没有正常的反序列化成功，但是程序执行没有出错，这也是通常容易忽略的。这个时候如果让BaseDog也实现Serializable接口，那么程序执行结果就正常了。同时也不需要什么无参构造器。所以关键就在于Serializable接口。

其中Serializable中这一段注释比较有意思

 * To allow subtypes of non-serializable classes to be serialized, the
 * subtype may assume responsibility for saving and restoring the
 * state of the supertype's public, protected, and (if accessible)
 * package fields.  The subtype may assume this responsibility only if
 * the class it extends has an accessible no-arg constructor to
 * initialize the class's state.  It is an error to declare a class
 * Serializable if this is not the case.  The error will be detected at
 * runtime. 
 *
 * During deserialization, the fields of non-serializable classes will
 * be initialized using the public or protected no-arg constructor of
 * the class.  A no-arg constructor must be accessible to the subclass
 * that is serializable.  The fields of serializable subclasses will
 * be restored from the stream. 

他说的意思大概就是如果一个类没有实现Serializable接口，但是想让他的子类序列化，那么父类就应该要有一个无参构造器，其中父类的属性会使用无参构造器初始化，子类实现了Serializable接口的会用子类序列化的属性。

## serialVersionUID

serialVersionUID这个是代表一个版本号，当序列化与反序列化的时候会对比类中的这个版本号，只有版本号相同的类才能反序列化成功，如果类中没有定义这个属性，那么在序列化的时候会自动生成一个,而且这个版本号必须是 static final long 。强烈建议在定义一个序列化类的时候就写上这个属性。

idea自动生成serialVersionUID,需要在 Preferences -> Editor -> Inspections -> Serialization issues -> Serialization class without "serialVersionUID" 这里打上勾，那么就可以通过 command + enter生成 serialVersionUID。

## transient关键字

这个关键字是用来修辞属性的，当一个属性用transient修饰之后，那么序列化与反序列化的时候就会忽视这个属性。
观察ArrayList 源码发现其中的数组使用transient,那么代表序列化的时候不序列化这个属性，但是结果表明我们在使用线性表的时候的确成功序列化里面的元素了
```java
transient Object[] elementData
```
原因在于ArrayList里面的writeObject方法，里面自定义了序列化的方式。因为并不是线性表中所有的数组格子我们都用了，比如当扩容的时候我们实际用到的可能是21个，但是由于扩容可能数组的长度是30个,那么我们序列化的时候肯定不希望将这30个都序列化，序列化无用的null没有意义。所以就自定义了序列化。

```java
/**
     * Save the state of the <tt>ArrayList</tt> instance to a stream (that
     * is, serialize it).
     *
     * @serialData The length of the array backing the <tt>ArrayList</tt>
     *             instance is emitted (int), followed by all of its elements
     *             (each an <tt>Object</tt>) in the proper order.
     */
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

改写Dog如下
```java

/**
 * Created by ruanwenjun on 2018/12/11
 */
public class Dog extends BaseDog implements Serializable {
    private String name;
    private int age;
    transient private Host host;


    public Dog(String hair, String name, int age, Host host) {
        super(hair);
        this.name = name;
        this.age = age;
        this.host = host;
    }

    @Override
    public String toString() {
        return "Dog{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", host=" + host +
                ", hair='" + hair + '\'' +
                '}';
    }
}
Dog{name='tom', age=1, host=null, hair='red'}
```
运行main方法，发现host为null,因为我们用tracsient修饰了，所以这个属性不会被序列化

再修改Dog
```java
/**
 * Created by ruanwenjun on 2018/12/11
 */
public class Dog extends BaseDog implements Serializable {
    private String name;
    private int age;
    transient private Host host;


    public Dog(String hair, String name, int age, Host host) {
        super(hair);
        this.name = name;
        this.age = age;
        this.host = host;
    }

    private void writeObject(java.io.ObjectOutputStream s)
            throws java.io.IOException {
        s.defaultWriteObject();
        s.writeObject(host);
    }

    private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();
        host = (Host) s.readObject();
    }

    @Override
    public String toString() {
        return "Dog{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", host=" + host +
                ", hair='" + hair + '\'' +
                '}';
    }
}

Dog{name='tom', age=1, host=Host{name='tom father'}, hair='red'}
```
执行main方法返回正常，而且仅仅只写writeObject方法的话返回host依然为null，猜测应该是transient修饰的方法既不序列化也不反序列化。






