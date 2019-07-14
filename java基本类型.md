### java基本类型

#### 两条准则

```
~如果对整数不指定类型，默认int类型，对小数不指定类型，默认为double
~基本类型从小到大，可以自动转换，但是由大到小，则需要强制类型转换
```

#### 所占字节数

```
byte：1个字节
char：2个字节
short：2个字节
int：四个字节
long：8个字节
float：4个字节
double：8个字节
char：Java中用 "\u四位十六进制的数字 (即使在注释中出现\u，后面如果
跟的不是4个16进制的数字，也会报错)"表示将字符转换成对应的unicode编 码；也可以用字符来赋值如： char c="\u0000" ，char的默认初始化值，unicode的null字符
```

#### 类型转换

int类型的字面常量转换成比int类型低的变量类型

```
    int a = 8;  //8是字面常量
    byte b = 9;  //9是字面常量
    char c = 9+5;//常量表达式
    short s = (short) (c+10); //变量表达式，需要显式强制转换
}
上面的代码是经过编译的，是正确的。b是byte类型，但b=9不需要显式地手动强制转换，这是因为9是字面常量，是由JVM自动完成。
  我们再来看一下`c=9+5`，c是char类型，9+5得到结果是int类型，但也不需要显式地手动强制转换。这是因为 9+5是常量表达式，所以在编译期间已经由编译器计算出结果了，即经过编译后，相当于 c=14，也是字面常量，所以可以隐式转换。同理，`short s = (short) (c+10);` 子所以不能隐式转换，就是因为表达式不是常量表达式，包含了变量，只能在运行期间完成，所以就要手动强制转换。
```

#### 整形字面常量隐式转换的限制：

```
byte b = 128;//编译错误，128超出byte类型所能表示的范围
byte c = (byte)128;//编译通过

```

##### 对于传参数时，必须要显式地进行强制类型转换，明确转换的类型

```
public static void main(String[] args) {
    shortMethod(8);//编译错误
    shortMethod((short)8); //编译通过
    longMethod(8);//编译通过，因为这是小类型变成大类型，是不需要强制类型转换的
}
public static void shortMethod(short c){
    System.out.println(c);
}
public static void longMethod(short l){
    System.out.println(l);
}
```

#### 复合运算符的隐式转换

复合运算符（+=、-=、\*=、/=、%=）是可以将右边表达式的类型自动强制转换成左边的类型

```
public static void main(String[] args) {
    int a = 8;  
    short s = 5; 
    s += a;
    s += a+5;   
}
```

`s+=a、s+=a+5`;的表达式计算结果都是int类型，但都不需要手动强制转换。其实，如果是反编译这段代码的class文件，会发现s+=a;，其实是被编译器处理成了s=(short)(s+a)

#### 特殊的char类型

char类型在基本类中是一个比较特殊的存在。这种特殊性在于char类型是一个无符号类型，所以char类型与其他基本类型不是子集与父集间的关系（其他类型都是有符号的类型）。也就是说，char类型与byte、short之间的转换都需要显式的强制类型转换（小类型自动转换成大类型失败）。同时，由于char类型是一个无符号类型，所以对于整形字面常量的隐式转换的限制，不仅包括字面常量数值的大小不能超出2个字节，还包括字面常量数值不能为负数

```
 byte b = 2;
 char c = 2;//编译通过
      c = 100000000000;//编译不通过，超出char类型的范围
 char d = -2//字面常量为负数，编译不通过
      d = (char)-100;//编译通过
 char f = (char)b; //编译通过，必须显式的强制类型转换
      f = b;//编译不通过，不能隐式转换
 int  i = c;//编译通过，可以不需要强制类型转换
 short s = (short) c;//编译通过，必须显式地强制类型转换
```

```
char类型与byte、short的相互转换，都需要显式地强类型制转换。
对于数值是负数的，都需要进行显式地强制类型转换，特别是在整形字面常量的隐式转换中。
char类型转换成int、long类型是符合 小类型转大类型的规则，即无需要强制类型转换。
```

#### 运算结果的类型

在Java中，一个运算结果的类型是与表达式中类型最高的相等

```
char cc = 5;
float dd = 0.6f+cc;//最高类型是float，运算结果是float
float ee = (float) (0.6d+cc);//最高类型是double，运算结果也是double
int aa = 5+cc;//最高类型是int，运算结果也为int
```

但是，**对于最高类型是byte、short、char的运算来说，则运行结果却不是最高类型，而是int类型**。c、d运算的最高类型都是char，但运算结果却是int，所以需要强制类型转换。

```
 byte b = 2;
 char a = 5;
 char c = (char) (a+b);//byte+char，运算结果的类型为int，需要强制类型转换
 int  e = a+b;//编译通过，不需要强制类型转换，可以证明是int
 char d = (char) (a+c);//char+char，
 short s1 = 5;
 short s2 = 6;
 short s3 =(short)s1+s2; 
```

#### 浮点数类型

存储的小数的数值可能是模糊值

```
public static void main(String[] args) {
    double d1 = 0.1;
    double d2 = 0.2;
    System.out.println(d1+d2 == 0.3);
    System.out.println(d1+d2);
}
false
 0.30000000000000004
```

很多小数都是无法准确地用浮点型表示，其实这是由 小数的十进制转成二进制的算法所决定的，十进制的小数要不断乘2，知道最后的结果为整数才是最后的二进制值，但这有可能怎么也得不到整数，所以最后得到的结果可能是一个 无限值 ，浮点型就无法表示了

  但是对于 整数 来说，在浮点数的有效范围内，则都是精确的。同样，也是由于转换算法：十进制的整数转成二进制的算法是不断对2求余数，所以 不会存在无限值的情况；

#### 浮点数的有效位及精度

浮点型所能表示的有效位是有限的，所以哪怕是整数，只要超出有效位数，也只能存储相似值，也就是该数值的最低有效位将会丢失,从而造精度丢失。

```
public static void main(String[] args) {
    int a = 3000000;
    int b = 30000000;
    float f1 = a;
    float f2 = b;
    System.out.println("3000000==3000001 "+(f1==f1+1));
    System.out.println("30000000==30000001 "+(f2==f2+1));
    System.out.println("3000000的有效二进制位数："+ Integer.toBinaryString(a).length());
    System.out.println("30000000的有效二进制位数："+ Integer.toBinaryString(b).length());
}
3000000 == 3000001  false
30000000 == 30000001  true
3000000的有效二进制位数： 22
30000000的有效二进制位数： 25
```

