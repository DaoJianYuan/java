##1.内部类相关

Author:老机长

createTime: 2017.07.12	
version:1.0

updateTime: 2017.07.15		
version:1.1

* 1.内部类作为外部类的成员变量

```
public class Outter {
    private String name = "老机长";
    private int age = 18;

    public static void main(String[] args) {
        Outter out = new Outter();
        out.new Inner().printInfo();
    }

    public class Inner {
        public void printInfo() {
            System.out.println("name = " + name + "\nage=" + age);
        }
    }
}
```

- 注释：非静态内部类的对象隐式持有外部类对象引用，外部类引用在内部类构造器中设置，编译器修改了所有内部类的构造器，添加了一个外部类引用参数。

- 问：为什么内部类可以访问外部类的成员变量

- 答：以下片段代码是Outter类的字节码文件内容，可以看到编译器自动给Outter添加了 access$xxx 的静态方法，成员内部类访问外部类对象即调用了外部类的静态方法。

```
Compiled from "Outter.java"
public class Outter {
  public Outter();
  public static void main(java.lang.String[]);
  static java.lang.String access$000(Outter);
  static int access$100(Outter);
}


Compiled from "Outter.java"
public class Outter$Inner {
  final Outter this$0;			//内部类持有外部类对象
  public Outter$Inner(Outter);	//编译器修改了内部类的构造方法
  public void printInfo();
}
```


* 2.局部内部类（匿名内部类也属于此范围）

我反手就是一个案例

```
public class Outter {
    public static void main(String[] args) {
        Outter out = new Outter();
        out.printInfo("老机长", 16, new Object());
    }

    public void printInfo(final String name, final int age, final Object obj) {
        abstract class Inner {
            abstract void printInfo();
        }

        class Inner2 {
            public void printInfo() {
                System.out.println("name = " + name + "\nage=" + age + "\nobj=" + obj);
            }
        }

        Inner inner = new Inner() {
                public void printInfo() {
                    System.out.println("name = " + name + "\nage=" + age + "\nobj=" + obj);
                }
            };

        inner.printInfo();

        new Inner2().printInfo();

        final Runnable runnable = new Runnable() {
                @Override
                public void run() {
                    System.out.print("Runnable" + "name = " + name + "\nage=" + age);
                }
            };

        new Thread() {
                @Override
                public void run() {
                    super.run();
                    runnable.run();
                }
            }.start();
    }
}
```
- 注释：局部内部类可以说对外部类世界完全隐藏起来，即使Outter外部类也不能访问Inner。除了调用的方法即Outter.printInfo()方法外，没有任何方法知道Inner类的存在。

- 局部内部类不需要也不允许添加访问修饰符，切记。切记。切记。

- 问：局部内部类的存在形式如何？？

- 答：查看以下代码片段，在编译期间，编辑器修改局部内部类的构造器方法，除了增加外部类引用，还会将局部变量拷贝到局部内部类的构造器，并存储在 val$xxx 变量中做备份，在内部类中用变量保存外部的局部变量副本。

- 为啥定义在方法内部的局部内部类调用局部变量需要加final关键字呢？？不加可以吗？
- 若不加final关键字，方法的局部变量可以在初始化内部类后进行修改，造成局部变量与内部类变量副本引用不一致，造成错乱，歧义等，不利于程序理解，故加final关键字禁止变量引用的修改。


> 内部类并不是直接调用方法传进来的参数，而是内部类将传进来的参数通过自己的构造器备份到了自己的内部，自己内部的方法调用的实际是自己的属性而不是外部类方法的参数。  
这样理解就很容易得出为什么要用final了，因为两者从外表看起来是同一个东西，实际上却不是这样，如果内部类改掉了这些参数的值也不可能影响到原参数，然而这样却失去了参数的一致性，因为从编程人员的角度来看他们是同一个东西，如果编程人员在程序设计的时候在内部类中改掉参数的值，但是外部调用的时候又发现值其实没有被改掉，这就让人非常的难以理解和接受，为了避免这种尴尬的问题存在，所以编译器设计人员把内部类能够使用的参数设定为必须是final来规避这种莫名其妙错误的存在。”
 (简单理解就是，拷贝引用，为了避免引用值发生改变，例如被外部类的方法修改等，而导致内部类得到的值不一致，于是用final来让该引用不可改变) [此段内容解释摘自于文末提供的博客链接中的一段话]


 
- 字节码文件欣赏

```
//以下属于外部类Outter字节码文件
Compiled from "Outter.java"
public class Outter {
  public Outter();
  public static void main(java.lang.String[]);
  public void printInfo(java.lang.String, int, java.lang.Object);
}

//局部抽象类Inner字节码文件
Compiled from "Outter.java"
abstract class Outter$1Inner {
  final Outter this$0;
  Outter$1Inner(Outter);	//此处字节码已修改
  abstract void printInfo();
}

//局部类自定义完整类Inner2字节码文件
Compiled from "Outter.java"
class Outter$1Inner2 {
  final java.lang.String val$name;
  final int val$age;
  final java.lang.Object val$obj;
  final Outter this$0;
  Outter$1Inner2();			//此处字节码未修改
  public void printInfo();
}

//实例化局部自定义抽象内部类Inner字节码文件
Compiled from "Outter.java"
class Outter$1 extends Outter$1Inner {
  final java.lang.String val$name;
  final int val$age;
  final java.lang.Object val$obj;
  final Outter this$0;
  Outter$1(Outter, java.lang.String, int, java.lang.Object);
  public void printInfo();
}

//局部内部类Runnable字节码文件
Compiled from "Outter.java"
class Outter$2 implements java.lang.Runnable {
  final java.lang.String val$name;
  final int val$age;
  final Outter this$0;
  Outter$2(Outter, java.lang.String, int);
  public void run();
}

//局部系统支持内部类Thread字节码文件
Compiled from "Outter.java"
class Outter$3 extends java.lang.Thread {
  final java.lang.Runnable val$runnable;
  final Outter this$0;
  Outter$3(Outter, java.lang.Runnable);
  public void run();
}
```

- 概念：什么叫完整性class  就是非抽象类 可直接实例化的类
- 问：草泥马，你不说编辑器修改了构造器么？怎么局部自定义完整类Inner2 javap看到的字节码内容没出现呢？？

- 答：这尼玛就尴尬了，从字节码文件好像确实无法验证到以上说法的正确性，这点确实有待考证，(我也很无奈，很绝望啊，好想哭)啪啪啪打脸。。。但是据我观察，貌似如果局部内部类定义是个完整性class 确实无法证实编译器修改了构造器，如果替换成抽象类，即可验证。
- 注：之前以为接口也属于此类情况，但是方法内局部不可定义接口，如果接口定义在外部类中，结论和局部抽象类是一致的，感兴趣的同学可以自行验证


## 资料来源
* [Java内部类的使用小结](http://android.blog.51cto.com/268543/384844)


