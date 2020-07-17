### 4 基础类型

#### 4.1 所有类型都从System.Object派生

    “运行时”要求每个类型最终都从System.Object类型派生。也就是说，以下两个类型的定义完全一
    致。

    //隐式派生自Object
    class Employee{

    }

    //显示派生自Object
    class Employee:System.Object{
        
    }

    由于所有类型最终都从System.Object派生，所以每个类型的对象都保证了一组最基本的方法。具
    体地说，System.Object类提供了如下公共实例方法：

        Equals          如果两个对象具有相同的值，就返回true
        GetHashCode     返回对象的值得哈希值
        ToString        默认返回类型的完整名称
        GetType         返回从Type派生的一个类型的实例，指出调用GetType的对象类型

    此外，从System.Object派生的类型能访问如下受保护的方法：

        MemberwiseClone 这个非虚方法创建类型的新实例，并将对象的实例字段设与this对象的实
                        例字段完全一致。返回对新实例的引用
        Finalize        在垃圾回收器判断对象应该作为垃圾被回收之后，在对象的内存被实际回
                        收之前，会调用这个虚方法。

    CLR要求所有对象都用new操作符来创建。
        Employee e=new Employee("ConstructorParam1");
    
    以下是new操作符所做的事情：

        1. 计算类型及其所有基类型（一直到System.Object,虽然它没有定义自己的实例字段）中
           定义的所有实例字段所需的字节数。堆上每个对象都需要一些额外的成员，包括“类型对
           象指针”和“同步块索引”。CLR利用这些成员管理对象。额外成员的字节数要计入对象大小
        
        2. 从托管堆中分配类型要求的字节数，从而分配对象的内存，分配的所有字节都设为零

        3. 初始化对象的“类型对象指针”和“同步块索引”成员

        4. 调用类型的实例构造器，传递在new调用中指定的实参（上例就是字符串“ConstructorP
           aram1”）。大多数编译器都在构造期中自动生成代码来调用基类构造器。每个类的构造器
           都负责初始化该类型定义的实例字段。最终调用System.Object的构造器，该构造器什么
           都不做，简单的返回。

    new执行了所有操作之后，返回指向新建对象一个引用。在前面的示例代码中，保存到了e中。

#### 4.2 类型转换

    CLR最重要的特性之一就是类型安全。在运行时，CLR总是知道对象的类型时什么。调用GetType方
    法即可知道对象的确切类型。由于它是非虚方法，所以一个类型不可能伪装成另一个类型。

    CLR允许将对象转换为它的（实际）类型或者他的任何基类型。C#不要求任何特殊语法即可将对象
    转换为他的任何基类型，因为向基类型的转换被认为是一种安全的隐式转换。然而将对象转换为它
    的某个派生类型时，C#要求开发人员只能进行进行显示转换，因为这种转换可能在运行时失败。

    使用C#的is和as操作符来转型

        is检查对象是否兼容于指定类型，返回Boolean值true或false。is永远不抛出异常。

            Object o=new Object();
            Boolean b1=(o is Object);
            Boolean b2=(o is Employee);

        is操作符通常像下面这这使用：
            if(o is Employee){
                Employee e=(Employee) o;
            }
        在上述代码中,CLR实际检查两次对象类型。is操作符首先核实o是否兼容于Employee类型。
        如果是，在if语句内部转型时CLR再次核实o是否引用一个Employee。这会对性能造成影响
        
        as操作符目的就是简化这种代码的写法，同时提升性能
        Employee e=o as Employee;
        if(e!=null){

        }

    注意：C#允许类型定义转换操作符方法。只有在使用转型表达式时才会调用这些方法，使用as或is
          操作符时永远不调用他们

#### 4.3 命名空间和程序集

    命名空间对相关的类型进行逻辑分组，开发人员可通过命名空间方便地定位类型。

    public sealed class Program{
        public static void Main(){
            System.IO.FileStream fs= new System.IO.FileStream(...);
            System.Text.StringBuilder sb= new System.Text.StringBuilder();
        }
    }
    像这样写代码很繁琐，应该有一种简单的方式直接引用FileStream和StringBuilder类型，减少
    打字量。C#编译器通过using指令提供这个机制。如下

    using System.IO;
    using System.Text;

    public sealed class Program{
        public static void Main(){
            FileStream fs= new FileStream(...);
            StringBuilder sb= new StringBuilder();
        }
    }

    对于编译器，命名空间的作用就是为类型名称附加以句点分割的符号，使名称变得更长，更可能具
    有唯一性。

    C#的using指令是可选的，它指示编译器尝试为类型名称附加不同的前缀，直至找到匹配项

    重要提示：
        CLR对“命名空间”一无所知。访问类型时，CLR需要知道类型的完整名称以及该类型的定义具
        体在哪个程序集中。这样“运行时”才能加载正确程序集，找到目标类型，并对其进行操作。

    在前面的代码中，编译器需要保证引用的每个类型都确实存在，而且代码以正确方式使用类型。如
    果编译器在源代码文件或者引用的任何程序集中找不到具有指定名称的类型，就是附加Using后面
    的前缀，再次寻找。

    编译器对待命名空间的方式存在潜在问题：
        可能两个或更多类型在不同命名空间中同名。这样可能会出现错误信息引用不明确

    为了解决潜在问题有以下方法：
        
        1. 显示告诉编译器要创建的具体是那个

        2. C# using指令的另一种形式允许为类型或命名空间创建别名

        3. c#编译器提供了名为外部别的功能

    在C#中，namespace指令的作用只是告诉编译器为源代码中出现的每个类型名称附加命名空间名称
    前缀，让程序员少打一些字。

    命名空间和程序集的关系：

        命名空间和程序集（实现类型的文件）不一定相关。特别是，同一个命名空间中的类型可能在
        不同的程序集中实现。例如，System.IO.FileStream类型在MSCorLib.dll程序集中实现，
        而System.IO.FileSystemWWatcher类型在System.dll程序集中实现。

        同一个程序集也可能包含不同命名空间中的类型。例如System.Int32和System.Text.Strin
        gBuilder类型都在MSCorLib.dll程序集中。

#### 4.4 运行时的相互关系

    接下来解释类型、对象、线程栈和托管堆在运行时的相互关系。此外还解释调用静态方法、实例方
    法和虚方法的区别。
                                                    线程栈
    void M1(){                                   ——————————————
        String name = "Joe";                    |      .       |  
        M2(name);                               |      .       |
        ...                                     |      .       |
        return                                  |              |
    }                                           |              |
                                                |              |
                                                |              |
                                                |              |
                                                 ——————————————

    上面展示了已加载CLR的一个Windows进程。该进程可能有多个线程。线程创建时会分配到1MB的栈
    栈空间用于向方法传递实参，方法内部定义的局部变量也在栈上。栈是从高位内存地址向低位内存
    地址构建。图中的线程以及执行了一些代码，栈上也有了一些数据。现在假定要调用M1方法。

                                                    线程栈
    void M1(){                                   ——————————————
        String name = "Joe";                    |      .       |  
        M2(name);                               |      .       |
        ...                                     |      .       |
        return                                  | name(string) |
    }                                           |              |
                                                |              |
                                                |              |
                                                |              |
                                                 ——————————————
    
    最简单的方法包含“序幕”代码，在方法开始工作前对其进行初始化；还包含“尾声”代码，在方法做
    完工作后对其进行清理，以便返回至调用者。M1开始执行时，他的序幕代码在线程栈上分配局部变
    量name的内存，如上图所示。

                                                    线程栈
    void M1(){                                   ——————————————
        String name = "Joe";                    |      .       |  
        M2(name);                               |      .       |
        ...                                     |      .       |
        return                                  | name(string) |
    }                                           |  s(string)   |
                                                |  [返回地址]   |
    void M2(string s){                          |              |
        Int32 length=s.Length;                  |              |
        Int32 tally;                            |              |
        ...                                      ——————————————
        return;
    }

    然后M1调用M2方法，将局部变量name作为实参传递。这造成name局部变量中的地址被压入栈。M2
    方法内部使用参数变量s标识栈位置。另外，调用方法时还会将“返回地址”压入栈。被调用的方法
    在结束之后应返回至该位置。

                                                    线程栈
    void M1(){                                   ——————————————
        String name = "Joe";                    |      .       |  
        M2(name);                               |      .       |
        ...                                     |      .       |
        return                                  | name(string) |
    }                                           |  s(string)   |
                                                |  [返回地址]   |
    void M2(string s){                          | length(Int32)|
        Int32 length=s.Length;                  | tally(Int32) |
        Int32 tally;                            |              |
        ...                                      ——————————————
        return;
    }
    M2方法开始执行时，他的序幕代码在线程栈中为局部变量length和tally分配内存，如上图。然后
    M2方法内部代码开始执行。最终M2抵达他的return语句，造成CPU的执行被设置成栈中的返回地址
    M2的栈帧展开。之后M1继续执行M2调用之后的代码，M1的栈帧将准确反映M1需要的状态。

    接下来围绕CLR讨论一下。假定有以下两个类定义：

    internal class Employee{
        public Int32 GetYearsEmployed() {...}
        public virtual String GetProgressReport {...}
        public static Employee Lookup(String name) {...}
    }

    internal seald class Manger : Employee{
        public override String GetProgressReport() {...} 
    }

    方法：
    --------------------------------
    |void M3 (){                   |
    |    Employee e;               |
    |    Int32 year;               | 
    |    e=new Manager();          | 
    |    e=Employee.Lookup("Joe"); |
    |    year=e.GetYearsEmployed();|
    |    e.GetProgressReport();    |
    |}                             | 
    --------------------------------

    线程栈：
    ----------
    |    .   |
    |    .   |
    |    .   |
    |        |
    |        |
    ----------

    堆
    -----------------------------------------
    |                                       |
    |                                       |
    |                                       |
    |                                       |
    |                                       |
    |                                       |
    |                                       |
    |                                       |
    -----------------------------------------

    Windows进程已启动，CLR已加载到其中，托管堆已初始化，而且已创建一个线程（连同它的1MB
    栈空间）。线程已经执行了一些代码，马上要调用M3方法。如上所示状态。M3包含代码演示了CLR
    是如何工作的。

    方法：
    --------------------------------
    |void M3 (){                   |
    |    Employee e;               |
    |    Int32 year;               | 
    |    e=new Manager();          | 
    |    e=Employee.Lookup("Joe"); |
    |    year=e.GetYearsEmployed();|
    |    e.GetProgressReport();    |
    |}                             | 
    --------------------------------

    线程栈：
    ----------
    |    .   |
    |    .   |
    |    .   |
    |        |
    |        |
    ----------

    堆
    -----------------------------------------
    |                       Manager类型对象   
    |                        类型对象指针     
    |                        同步索引块       
    |                        静态字段         
    |                       --------------   
    |                     GetProgressReport
    |
    |                       
    |                       Employee类型对象
    |                         类型对象指针
    |                         同步索引块
    |                       ---------------
    |                     GetYearsEmployed
    |                     GetProgressReport
    |                     Lookup
    -----------------------------------------

    JIT编译器将M3的IL代码转换成本机CPU指令时，会注意到M3内部引用的所有类型，包括Employee
    Int32,Manager以及String。这时CLR要确认定义了这些类型的所有程序集都已经加载。然后，利
    用程序集的元数据，CLR提取这些类型有关的信息，创建一些数据结构来表示类型本身。如上展示
    了为Employee和Manager类型对象使用的数据结构。由于在现场调用M3前已经执行了一些代码，所
    以假设Int32和String类型对象已经创建好了，所以图中没显示他们

    堆上所有对象都包含两个额外的成员：类型对象指针和同步块索引。如上所示Manager和Employee
    类型对象都有这两个成员。定义类型时，可在类型内部定义静态数据字段。为这些静态数据字段提
    供支援的字节在类型对象自身中分配。每个类型对象最后都包含一个方法表。在方法表中，类型定
    义的每个方法都有对应的记录项。

    方法：
    --------------------------------
    |void M3 (){                   |
    |    Employee e;               |
    |    Int32 year;               | 
    |    e=new Manager();          | 
    |    e=Employee.Lookup("Joe"); |
    |    year=e.GetYearsEmployed();|
    |    e.GetProgressReport();    |
    |}                             | 
    --------------------------------

    线程栈：
    ---------------
    |      .      |
    |      .      |
    |      .      |
    | e(Employee) |
    | year(int32) |
    ---------------

    堆
    -----------------------------------------
    |                       Manager类型对象   
    |                        类型对象指针     
    |                        同步索引块       
    |                        静态字段         
    |                       --------------   
    |                     GetProgressReport
    |
    |                       
    |                       Employee类型对象
    |                         类型对象指针
    |                         同步索引块
    |                       ---------------
    |                     GetYearsEmployed
    |                     GetProgressReport
    |                     Lookup
    -----------------------------------------

    当CLR确认方法需要的所有类型对象都已创建，M3的代码编译之后，就允许线程执行M3的本机代码
    M3的序幕代码执行时必须在线程栈中为局部变量分配内存，如上所示。作为方法序幕代码的一部分
    CLR会自动将所有局部变量初始化为null或0

    方法：
    --------------------------------
    |void M3 (){                   |
    |    Employee e;               |
    |    Int32 year;               | 
    |    e=new Manager();          | 
    |    e=Employee.Lookup("Joe"); |
    |    year=e.GetYearsEmployed();|
    |    e.GetProgressReport();    |
    |}                             | 
    --------------------------------

    线程栈：
    ---------------
    |      .      |
    |      .      |
    |      .      |
    | e(Employee) |
    | year(int32) |
    ---------------

    堆
    ----------------------------------------------------------------
    |       Manager对象                              Manager类型对象   
    |       类型对象指针   ------------------------>   类型对象指针     
    |       同步索引块                                 同步索引块       
    |        实例字段                                  静态字段         
    |                                               --------------   
    |                                               GetProgressReport
    |
    |                       
    |                                               Employee类型对象
    |                                                类型对象指针
    |                                                同步索引块
    |                                               ---------------
    |                                               GetYearsEmployed
    |                                               GetProgressReport
    |                                               Lookup
    -----------------------------------------------------------------

    M3执行代码构造一了Manager对象。这造成在托管堆创建Manager类型的一个实例（也就是一个M
    anager对象），如上所示。可以看出和所有对象一样，Manager对象也有类型对象指针和同步块索
    引。该对象还包含必要的字节来容纳Manager类型定义的所有实例数据字段，以及容纳由Manager
    的任何基类定义的所有实例字段。任何时候在堆上新建对象，CLR都自动初始化内不得“类型对象
    指针”成员来引用和对象对应的类型对象。此外，在调用类型构造器之前，CLR会先初始化同步块索
    引，并将对象的所有实例字段设为null或0。new操作符返回Manager对象的内存地址，该地址保存
    到变量e中（e在栈上）

    方法：
    --------------------------------
    |void M3 (){                   |
    |    Employee e;               |
    |    Int32 year;               | 
    |    e=new Manager();          | 
    |    e=Employee.Lookup("Joe"); |
    |    year=e.GetYearsEmployed();|
    |    e.GetProgressReport();    |
    |}                             | 
    --------------------------------

    线程栈：
    ---------------
    |      .      |
    |      .      |
    |      .      |
    | e(Employee) |
    | year(int32) |
    ---------------

    堆
    ----------------------------------------------------------------
    |       Manager对象                              Manager类型对象   
    |       类型对象指针   ------------------------>   类型对象指针     
    |       同步索引块                 ^                同步索引块       
    |        实例字段                 .                 静态字段         
    |                                .               --------------   
    |                                .              GetProgressReport
    |       Manager对象              .
    |       类型对象指针   -----------           
    |       同步块索引                                Employee类型对象
    |        实例字段                                 类型对象指针
    |                                                同步索引块
    |                                               ---------------
    |                                               GetYearsEmployed
    |                                               GetProgressReport
    |                                               Lookup---->JIT编译的代码
    -----------------------------------------------------------------

    M3的下一行代码调用Employee的静态方法Lookup。调用静态方法时，CLR会定位与定义静态方法
    的类型对应的类型对象。然后JIT编译器在类型对象的方法表中查找与被调用方法对应的记录项，
    对方法进行JIT编译，在调用JIT编译好的代码。假定Lookup方法在堆上构造一个新的Manager对
    象，返回该对象的地址。保存到局部变量e中。如上所示。e不在引用第一个Manager对象。事实上
    他是未来垃圾回收的主要目标。

    M3的下一行代码调用Employee的非虚实例方法GetYearsEmployed。调用非虚实例方法时,JIT编
    译器会找到与“发出调用的那个变量（e）的类型（Employee）”对应的类型对象。这时的变量e被
    定义成一个Employee。如果Employee类型没有定义正在调用的那个方法，JIT编译器会回溯类层
    次结构（一直回溯到Object）,之所以能回溯，是应为每个类型对象都有一个字段引用了他的基类
    型。假设该方法返回5，则这个证书保存到局部变量year中。

    M3的下一行代码调用了Employee的虚实例方法GetProgressReport。调用虚实例方法时JIT编译
    器要在方法中生成一些额外的代码；方法每次调用都会执行这些代码。这些代码首先检查发出调用
    的变量，并跟随地址来到发出调用的对象。变量e当前引用的是代表“Joe”的Manager对象。然后，
    代码接茬对象内部的“类型对象指针”成员，该成员指向了对象的实际类型。然后，代码在类型对象
    的方法表中查找引用了被调用方法的记录项，对方法进行JIT编译。

    进一步探讨CLR内部发生的事情

        注意Employee和Manager类型对象都包含“类型对象指针”成员。这是由于类型对象本质上也
        是对象。CLR创建类型对象时，必须初始化这些成员。

        初始化什么呢？

            CLR开始在一个进程中运行时，会立即为MSCorLib.dll中定义的Systerm.Type类型创建
            一个特殊的类型对象。Employee和Manager类型对象都是该类型的“实例”。因此，他们
            的类型对象指针成员会初始化成对System.Type类型对象的引用。










