对任何“活”的对象，一定能最终追溯到其存活在堆栈或静态存储区之中的引用

自动初始化的进行将在构造器被调用之前发生



**访问权限**

Java提供了访问权限修饰词，以供类库开发人员向客户端程序员指明哪些是可用的，哪些是不可用的。这样的话，类库的开发者就可以对代码进行修改，并确保客户代码不会因为这些改动而收到影响。把变动的事物与保持不变的事物区分开发

对于类的访问权限，仅有两个选择：包访问权限或public；但一个内部类可以是private或是protected，但那是一个特例



控制对成员的访问权限有两个原因：

1. 为了使用户不要触碰那些他们不该触碰的部分，这些部分对于类内部的操作是必要的，但是它并不属于客户端程序员所需接口的一部分。这样客户端程序员就可以清楚的看到什么对他们重要，什么是他们可以忽略的。这样简化了他们对类的理解
2. 让类库设计者可以更改类的内部工作方式，而不必担心这样会对客户端程序员产生重大的影响。访问控制权限可以确保不会有任何客户端程序员依赖于某个类的底层实现的任何部分



**组合和继承**

到底是该用组合还是用继承，一个最清晰的判断办法就是是否需要从新类向基类进行向上转型。如果必须向上转型，则继承是必要的；但如果不需要，则应当好好考虑是否需要继承



**final**

不想做改变可能出于两种原因：设计或效率



**多态**

域、静态方法、私有方法：

+ 任何域访问操作都将由编译器解析，因此不是多态的
+ 静态方法是与类，而并非与单个对象相关联的
+ 私有方法被自动认为是final方法，而且对导出类是屏蔽的，因此不能被重载



基类的构造器总是在导出类的构造过程中被调用，而且按照继承层次逐渐向上链接，以使每个基类的构造器都能得到调用。这样做是有意义的，因为构造器具有一项特殊任务：`检查对象是否被正确地构造`。导出类只能访问它自己的成员，不能访问基类中的成员（基类成员通常是private类型）。只有基类的构造器才具有恰当的知识和权限来对自己的元素进行初始化。因此，必须令所有构造器都得到调用，否则就不可能正确构造完整对象。这正是编译器为什么要强制每个导出类部分都必须调用构造器的原因



构造器调用顺序：

1. 调用基类构造器。这个步骤会不断地反复递归下去，首先是构造器这种层次结构的根，然后是下一层导出类，等等，直到最底层的导出类
2. 按声明顺序调用成员的初始化方法
3. 调用导出类构造器的主体



**构造器内部的多态方法的行为**

如果要调用构造器内部的一个动态绑定方法，就要用到那个方法的被覆盖后的定义。然而，这个调用的效果可能相当难于预料，因为被覆盖的方法在对象被完全构造之前就会被调用。这可能会造成一些难于发现的隐藏错误

从概念上讲，构造器的工作实际上是创建对象。在任何构造器内部，整个对象可能只是部分形成——我们只知道基类对象已经进行初始化。如果构造器只是在构建对象过程中的一个步骤，并且该对象所属的类是从这个构造器所属的类导出的，那么导出部分在当前构造器正在被调用的时刻仍旧是没有被初始化。然而，一个动态绑定的方法调用却会向外深入到继承层次结构内部，它可以调用导出类里的方法。如果我们是在构造器内部这样做，那么就可能会调用某个方法，而这个方法所操纵的成员可能还未进行初始化——这肯定会招致灾难

~~~
class Glyph {
    void draw() {
        System.out.println("Glyph.draw");
    }
    public Glyph() {
        System.out.println("Glyph before");
        draw();
        System.out.println("Glyph after");
    }
}

class RoundGlyph extends Glyph {
    private int radius = 1;
    public RoundGlyph(int radius) {
        this.radius = radius;
        System.out.println("RoundGlyph：" + radius);
    }
    @Override
    void draw() {
        System.out.println("RoundGlyph draw：" + radius);
    }
}
public class PolyConstructors {
    public static void main(String[] args) {
        Glyph glyph = new RoundGlyph(5);
    }
}
// output
Glyph before
RoundGlyph draw：0
Glyph after
RoundGlyph：5
~~~



编写构造器时有一条有效的准则：“用尽可能简单的方法使对象进入正常状态；如果可以的话，避免调用其他方法”。在构造器内唯一能够安全调用的那些方法是基类中的final方法（也适用于private方法，它们自动属于final方法）。这些方法不能被覆盖，因此就不会出现上述令人惊讶的问题。

----

**接口**

接口方法默认是public，子类override的方法也必须定义为public，否则默认为包访问权限，这样在方法被继承的过程中，其可访问权限就被降低了，这是Java编译器所不允许的

----

**内部类**

内部类自动拥有对其外围类所有成员的访问权。当某个外围类对象创建了一个内部类对象时，此内部类对象必定会秘密地捕获一个指向那个外围类 对象的引用。然后，当你访问此外围类的成员时，就是用那个引用来选择外围类的成员。 

在拥有外部类对象之前是不可能创建内部类对象的。这时因为内部类对象会暗暗地连接到创建它的外部类对象上。但是，如果你创建的是嵌套类（静态内部类），那么它就不需要对外部类对象的引用



如果不需要内部类对象与其外围类对象之间有联系，那么可以将内部类声明为static。这通常称为嵌套类。普通的内部类对象隐式地保存了一个引用，指向创建它的外部类对象。然而，当内部类是static的时，就不是这样了。嵌套类意味着：

+ 要创建嵌套类的对象，并不需要其外部类的对象
+ 不能从嵌套类的对象中访问非静态的外部类对象
+ 普通的内部类不能有static数据和static字段，也不能包含嵌套类；但嵌套类可以



----



**class对象**

所有的类都是在对其第一次使用时，动态加载到JVM中。当程序创建第一个对类的静态成员的引用时，就会加载这个类。这个证明构造器也是类的静态方法，即使在构造器之前并没有使用static关键字。因此，使用new操作符创建类的新对象也会被当作对类的静态成员的引用



Class.forName("Gum")：有副作用，如果类Gum还没有被加载就加载它，在加载的过程中，Gum会初始化



**类字面常量**

当使用“.class”来创建对Class对象的引用时，不会自动地初始化该Class对象

初始化被延迟到了对静态方法（构造器隐式地是静态的）或者非常数静态域进行首次引用时才执行

如果一个static final值是“编译器常量”，那么这个值不需要对该类进行初始化就可以被读取



**泛化的Class引用**

Class<?>的好处是它表示你并非是碰巧或者由于疏忽，而使用了一个非具体的类引用，你就是选择了非具体的版本



**instanceof与Class的等价性**

instanceof保持了类型的概念，它指的是“你是这个类吗，或者你是这个类的派生类吗？”；而如果用==比较实际的Class对象 ，就没有考虑继承——它或者是这个确切的类型，或者不是



**反射：运行时的类型信息**

RTTI（Run-Time Type Information)：在编译时，编译器必须知道所有通过RTTI来处理的类

RTTI与反射的真正区别只在于：对RTTI来说，编译器在编译时打开和检查.class文件；而对于反射机制来说，.class文件在编译时是不可获取的，所以是在运行时打开和检查.class文件



Class.forName()生成的结果在编译时是不可知的，因此所有的方法特征签名信息都是在执行时被提取出来的

反射机制提供了足够的支持，使得能够创建一个在编译时完全未知的对象，并调用此对象的方法



**泛型**

实现了参数化类型，解耦类或方法与所使用的类型之间的约束

是一种方法，通过它可以编写出更“泛化”的代码，这些代码对于它们能够作用的类型具有更少的限制，因此单个的代码段可以应用到更多的类型上

类型参数：暂时不指定类型，而是稍后再决定具体使用什么类型



**擦除**

擦除主要的正当理由是从非泛化代码到泛化代码的转变过程，以及在不破坏现有类库的情况下，将泛型融入Java语言。擦除使得现有的非泛型客户端代码能够在不改变的情况下继续使用，直至客户端准备好用泛型重写这些代码



**混型**

混合多个类的能力，以产生一个可以表示混型中所有类型的类

产生自泛型的类包含所有感兴趣的方法，但是由使用装饰器所产生的对象类型是最后被装饰的类型。也就是说，尽管可以添加多个层，但是最后一层才是实际的类型，因此只有最后一层的方法是可视的，而混型的类型是所有被混合到一起的类型。因此对于装饰器来说，其明显的缺陷是它只能有效地工作于装饰中的一层（最后一层），而混型方法显然会更自然一些。因此，装饰器只是对由混型提出的问题的一种局限的解决方案



**潜在类型机制**

潜在类型机制、结构化类型机制、鸭子类型机制：如果它走起来像鸭子，并且叫起来也像鸭子，那么你就可以将它当作鸭子对待

我不关心我在这里使用的类型，只要它具有这些方法即可

泛型代码典型地将在泛型类型上调用少量方法，而具有潜在类型机制的语言只要求实现某个方法子集，而不是某个特定类或接口，从而放松了这种机制（并且可以产生更加泛化的代码）。正由于此，潜在类型机制使得你可以横跨类继承结构，调用不属于某个公共接口的方法。因此，实际上一段代码可以声明：“我不关心你是什么类型，只要你可以speak()和sit()即可。”由于不要求具体类型，因此代码就可以更加泛化



**Read和Writer**

设计Reader和Writer继承层次结构主要是为了国际化。老的I/O流继承层次结构仅支持8为字节流，并且不能很好地处理16为的Unicode字符。由于Unicode用于字符国际化（Java本身的char也是16为的Unicode），所以添加Reader和Writer继承层次结构就是为了在所有的I/O操作中都支持Unicode



----

**注解**

> 注解（也被称为元数据）为我们在代码中添加信息提供了一种形式化的方法，使我们可以在稍后某个时刻非常方便地使用这些数据

**默认值限制**

编译器对注解元素的默认值有些过分挑剔。首先，元素不能有不确定的值。也就是说，元素必须要么具有默认值，要么在使用注解时提供元素的值

其次，对于非基本类型的元素，无论是在源代码中声明时，或是在注解接口中定义默认值时，都不能以null作为其值



**快捷方式**

如果注解中定义名为value的元素，并且在应用该注解的时候，如果该元素是唯一需要赋值的一个元素，那么此时无需使用名-键对的这种语法，而只需在括号内给出value元素所需的值即可



----



**异常情形**

异常情形（exception condition）是指阻止当前方法或作用域继续执行的问题。把异常情形与普通问题相区分很重要，所谓的普通问题是指，在当前环境下能得到足够的信息，总能处理这个错误。而对于异常情形，就不能继续下去了，因为在当前环境下无法获得必要的信息来解决问题。你所能做的就是从当前环境跳出，并且把问题提交给上一级环境。这就是抛出异常时所发生的事情



> 事务的基本保障是我们所需的在分布式计算中的异常处理。事务是计算机中的合同法，如果出了什么问题，我们只需要放弃整个计算



长久以来，尽管程序员们使用的操作系统支持恢复模型的异常处理，但他们最终还是转向使用类似“终止模型”的代码，并且忽略恢复行为。所以虽然恢复模型开始显得很吸引人，但不是很实用。其中的原因可能是它所导致的耦合：恢复性的处理程序需要了解异常抛出的地点，这势必要包含依赖于抛出位置的非通用性代码。这增加了代码编写和维护的困然，对于异常可能会从许多地方抛出的大型程序来说，更是如此



printStackTrace()：从方法调用处直到异常抛出处的方法调用序列；所提供的信息可以通过getStackTrace()方法来直接访问，这个方法将返回一个由栈轨迹中的元素所构成的数组，其中每一个元素都表示栈中的一帧。元素0是栈顶元素，并且是调用序列中的最后一个方法调用（这个Throwable被创建和抛出之处）。数组中的最后一个元素和栈底是调用序列中的第一个方法调用



+ 重抛异常会把异常抛给上一级环境中的异常处理程序，同一个try块的后续catch子句将被忽略
+ 如果只是把当前异常对象重新抛出，那么printStackTrace()方法显示的将是原来异常抛出点的调用栈信息，而并非重新抛出点的信息。要想更新这个信息，可以调用fillStackTrace()方法。有关原来异常发生点的信息会丢失，剩下的是与新的抛出点有关的信息



通过强制派生类遵循基类方法的异常说明，对象的可替换性得到了保证

异常限制对构造器不起作用，派生类的构造器可以抛出任何异常，而不必理会基类构造器所抛出的异常。然而，因为基类构造器必须以这样或那样的方式被调用，派生类构造器的异常说明必须包含基类构造器的异常说明

尽管在继承过程中，编译器会对异常说明做强制要求，但异常说明本身并不属于方法类型的一部分，方法类型是由方法的名字与参数的类型组成的。因此，不能基于异常说明来重载方法。此外，一个出现在基类的异常说明中的异常，不一定会出现在派生类方法的异常说明里。这点同继承的规则明显不同，在继承中，基类的方法必须出现在派生类里，换句话说，在继承和覆盖的过程中，某个特定方法的“异常说明的接口”不是变大了而是变小了——这恰好和类接口在继承时的情形相反




