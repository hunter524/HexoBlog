---
title: JAVA反射的初步理解和使用（一）
tags: JAVA核心类库、API、基础知识
date: 2017/2/24 12:50:26
---
# 前言
笔者才开始作为一个Android的入门级开发者，也是不太理解反射这个东西的。才入门是总是记得一句真理“面向对象编程，万物皆对象”。当然这句话在了解反射之后依旧发现他还是真理。首先我们知道对象是由类生成的，那我们用什么来描述“类”这个东西？类这个东西里面由包含字段，我们又用什么去描述“字段”这个东西？类里面又包含方法，我们又用什么来描述“方法”这个东西？字段、类、方法 又有各种访问控制符和限定符，访问控制符如 PUBLIC PROTECTED PRIVATE等，限定符如ABSTRACT STATIC  FINAL TRANSIENT VOLATILE SYNCHRONIZED NATIVE STRICT INTERFACE 这些又如何去描述？在大多数时候我们不该用反射，因为反射的性能相对较低。同时在大多数时候我们也不会用到反射，因为没必要。
笔者为什么会用反射，也是由于需求和神奇的接口导致的，也是因为笔者较为懒惰。为了脱离实际业务假设一种场景 我们使用两个接口获取一个数据对象，这里我们假设为对象为Abean，第一个接口返回的bean数据只有name和sex属性，第二个接口会返回该bean的其他数据，如果第二个接口返回的数据第一个接口也有使用第二个接口的数据，因为它更准确。但是我们此时又需要一个完整的bean数据，即要求所有数据都有且都是准确可靠的！
笔者首先想到的便是 挨个赋值当然这也是最简单高效的方法，然而笔者屁颠屁颠的挨个属性去get值再set完之后！一个晴天霹雳来了。又出现了一个Bbean，难道笔者还要去挨个赋值吗？写这些东西不是很没意义吗？如果又出现第三个CBean，第四个DBean....怎么办？就这么写set get 挨个赋值吗？如果每个Bean有十几个属性要赋值呢？那不是要写十几条get set语句去赋值？可能有人会问为什么不改接口呢？让一个Bean的数据一次性返回！笔者总是善意的理解为接口也是有难处的，改不了！那么只能自力更生去处理这样一个看似简单的事情了。
## CodeTalk

Abean的定义(省略set get toString方法)
```
public class Abean {
  private String name ;
  private String sex ;
  private String year;
  private String skinColor;
  .
  .
  .
}
```

利用反射赋值的核心逻辑(其实这个copyBean方法可以支持任意逻辑)
```
public class ReflectApiDemo {

    /**
     * copy src 2 des
     * @param src
     * @param des
     */
   private static Object copyBean(Object src, Object des) {
        Class desClass = des.getClass();
        String desClassName = desClass.getName();
        Class srcClass = src.getClass();
        String srcClassName = srcClass.getName();
        //判断src 和 des是否是同一个类的对象
        if (!srcClassName.equals(desClassName)) {
            throw new IllegalArgumentException("src and des must the same class!");
        }
        Field[] fields = desClass.getDeclaredFields();
        for (Field field : fields) {
            try {
                //                设置访问权限
                //如果不设置访问权限则默认不可以获取private字段值
                field.setAccessible(true);
                Object srcV = field.get(src);
                if (srcV != null) {
                    field.set(des, srcV);
                }
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
        }
        return des;
    }

    public static Abean copyAbean(Abean src,Abean des){
        return (Abean)copyBean(src,des);
    }
}
```
模拟前言问题中的返回数据对上面的方法进行测试
```
public static void main(String[] args){
    Abean abean1 = new Abean();
    abean1.setName("abean1 name!");
    abean1.setSex("abean1 male!");

    Abean abean2 = new Abean();
    abean2.setSex("abean2 female!");
    abean2.setYear("abean2 year!");

    ReflectApiDemo.copyAbean(abean2,abean1);

    DemoUtil.println(abean1.toString());

}
}
```
## 前言End
至此我们已经可以完美解决前言中的问题，且对任意类型的Bean都可以适用。上述的代码实现了把src这个对象中的字段值不为空的字段的值赋值到des的相同的字段中。这样笔者再也不用去多一个bean就去重复写一遍get set赋值语句了。
# 反射概述
## 什么是反射
通常我们所说的反射这种技术为什么他叫反射？说起反射这种技术为什么叫反射，可能因为反射相关的API都位于JDK的java.lang.reflect包下，reflect Google翻译一下。然而我们可以换一种理解方式，日常生活中我们所说的反射必须依赖一个镜子，我们自己每天照镜子能在镜子里面看见自己是因为镜子反射了你的影像。你才能看清自己的真是面目。对于JAVA的对象，他也会去照镜子，它的镜子便是各种JVM虚拟机，对象也能在镜子里面看清自己，它看到的那便是类。那么反射这种技术我们就可以理解为在JVM中运行的对象可以通过这种方式去看清自己。
## 反射相关的类梳理
说到反射相关的类，我们先梳理一下类里面可能包含哪些元素？一个类的实例即一个对象当然会有属性值即字段。至少有一个构造方法，当然你可以不写但是它也存在一个默认的构造方法。一个对象当然也会有各种行为，那么类里面便会有各种方法。那么我们现在要表示一个类需要哪些东西？我们需要 字段Field，方法Method，构造方法Constructor。当然我们还需要一个类，因为反射第一步通常拿到的便是Class这个类的对象。在JAVA中什么都是对象，我们通过反射拿到了任意一个对象的类或者直接拿到一个类，那么如何表示它？用对象！那这个类用什么类表示？Class。其实说反射我们拿到类这并不准确，其实我们拿到的还是对象。只不过这个对象能够表示一个类。正如前言所述“万物皆对象”。
那么与我们相关的便有了Class Field Method Constructor 四个类。
那么字段 类 方法 构造方法又都可以被注解标记。那么又多了一个类 Annotation，其实它并不是类只是一种接口，关于注解本篇不做详细解释。我们暂且理解为他是一种标记可以存在于类 方法 字段 构造方法上面的一种符号。最常见的一种注解：
```
@Override
```
注解的类容不详细展开，我们现在只需要知道要表示一个类其也是最基本的元素。除了注解，对于Class Field Method Constructor我还可以用各种限定符进行修饰那么如前言所述的 PUBLIC PROTECTED PRIVATE这些访问权限控制符我们又如何去表示呢？除了这些访问权限的控制修饰符还有很多其他修饰符又怎么表示？说到修饰符，此处不再赘述又一个隆重登场即Modifier。
以下是所有的修饰符：
```
/**
    * The {@code int} value representing the {@code public}
    * modifier.
    */
   public static final int PUBLIC           = 0x00000001;

   /**
    * The {@code int} value representing the {@code private}
    * modifier.
    */
   public static final int PRIVATE          = 0x00000002;

   /**
    * The {@code int} value representing the {@code protected}
    * modifier.
    */
   public static final int PROTECTED        = 0x00000004;

   /**
    * The {@code int} value representing the {@code static}
    * modifier.
    */
   public static final int STATIC           = 0x00000008;

   /**
    * The {@code int} value representing the {@code final}
    * modifier.
    */
   public static final int FINAL            = 0x00000010;

   /**
    * The {@code int} value representing the {@code synchronized}
    * modifier.
    */
   public static final int SYNCHRONIZED     = 0x00000020;

   /**
    * The {@code int} value representing the {@code volatile}
    * modifier.
    */
   public static final int VOLATILE         = 0x00000040;

   /**
    * The {@code int} value representing the {@code transient}
    * modifier.
    */
   public static final int TRANSIENT        = 0x00000080;

   /**
    * The {@code int} value representing the {@code native}
    * modifier.
    */
   public static final int NATIVE           = 0x00000100;

   /**
    * The {@code int} value representing the {@code interface}
    * modifier.
    */
   public static final int INTERFACE        = 0x00000200;

   /**
    * The {@code int} value representing the {@code abstract}
    * modifier.
    */
   public static final int ABSTRACT         = 0x00000400;

   /**
    * The {@code int} value representing the {@code strictfp}
    * modifier.
    */
   public static final int STRICT           = 0x00000800;

   // Bits not (yet) exposed in the public API either because they
   // have different meanings for fields and methods and there is no
   // way to distinguish between the two in this class, or because
   // they are not Java programming language keywords
   static final int BRIDGE    = 0x00000040;
   static final int VARARGS   = 0x00000080;
   static final int SYNTHETIC = 0x00001000;
   static final int ANNOTATION  = 0x00002000;
   static final int ENUM      = 0x00004000;
   static final int MANDATED  = 0x00008000;
   ```
   每一种修饰符能修饰的元素也是有限制的，比如我们不能用一个static去修饰一个方法的参数，我们也不能用native去修饰一个类，或者一个类的字段。每一种元素能使用哪些修饰符JAVA也是规定好的，同时在反射相关的Modifier类也做了说明。
   ```
   /**
     * The Java source modifiers that can be applied to a class.
     * @jls 8.1.1 Class Modifiers
     */
    private static final int CLASS_MODIFIERS =
        Modifier.PUBLIC         | Modifier.PROTECTED    | Modifier.PRIVATE |
        Modifier.ABSTRACT       | Modifier.STATIC       | Modifier.FINAL   |
        Modifier.STRICT;

    /**
     * The Java source modifiers that can be applied to an interface.
     * @jls 9.1.1 Interface Modifiers
     */
    private static final int INTERFACE_MODIFIERS =
        Modifier.PUBLIC         | Modifier.PROTECTED    | Modifier.PRIVATE |
        Modifier.ABSTRACT       | Modifier.STATIC       | Modifier.STRICT;


    /**
     * The Java source modifiers that can be applied to a constructor.
     * @jls 8.8.3 Constructor Modifiers
     */
    private static final int CONSTRUCTOR_MODIFIERS =
        Modifier.PUBLIC         | Modifier.PROTECTED    | Modifier.PRIVATE;

    /**
     * The Java source modifiers that can be applied to a method.
     * @jls8.4.3  Method Modifiers
     */
    private static final int METHOD_MODIFIERS =
        Modifier.PUBLIC         | Modifier.PROTECTED    | Modifier.PRIVATE |
        Modifier.ABSTRACT       | Modifier.STATIC       | Modifier.FINAL   |
        Modifier.SYNCHRONIZED   | Modifier.NATIVE       | Modifier.STRICT;

    /**
     * The Java source modifiers that can be applied to a field.
     * @jls 8.3.1  Field Modifiers
     */
    private static final int FIELD_MODIFIERS =
        Modifier.PUBLIC         | Modifier.PROTECTED    | Modifier.PRIVATE |
        Modifier.STATIC         | Modifier.FINAL        | Modifier.TRANSIENT |
        Modifier.VOLATILE;

    /**
     * The Java source modifiers that can be applied to a method or constructor parameter.
     * @jls 8.4.1 Formal Parameters
     */
    private static final int PARAMETER_MODIFIERS =
        Modifier.FINAL;

    /**
     *
     */
    static final int ACCESS_MODIFIERS =
        Modifier.PUBLIC | Modifier.PROTECTED | Modifier.PRIVATE;
   ```
   至此我们需要表示一个类我需要 Class Field Method Constructor Annotation Modifier这四个类，直接表示一个类的当然还是Class这个类。Field Method Constructor这些都是包含于Class类中的。一个类可以有0-N个Field、Method，但是必定会有一个Constructor。对于Class Field Method Constructor的修饰至少有一个修饰符，这些元素也是可以被注解标记的。注解对于这些元素的修饰是可以有，也可以不存在的。
   至此我们已经可以完完整的表示一个类了，然而在JAVA的世界，类和对象是最主要的，类实例化产生对象。然而还存在一些元素是不可以被实例化的如Interface、Enum、Annotation、数组和各种基础数据类型它们又怎么表示？那么笔者注意瞄了一眼Class类的注释。
   ```
* Instances of the class {@code Class} represent classes and
* interfaces in a running Java application.  An enum is a kind of
* class and an annotation is a kind of interface.  Every array also
* belongs to a class that is reflected as a {@code Class} object
* that is shared by all arrays with the same element type and number
* of dimensions.  The primitive Java types ({@code boolean},
* {@code byte}, {@code char}, {@code short},
* {@code int}, {@code long}, {@code float}, and
* {@code double}), and the keyword {@code void} are also
* represented as {@code Class} objects.
*
* <p> {@code Class} has no public constructor. Instead {@code Class}
* objects are constructed automatically by the Java Virtual Machine as classes
* are loaded and by calls to the {@code defineClass} method in the class
* loader.
*For example, the type of {@code String.class} is {@code
 * Class<String>}
   ```
