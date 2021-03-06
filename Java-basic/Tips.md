##### 1.用奇判断不用偶判断

```java
num % 2 == 0 ? "偶数" : "奇数";
```

`%`运算等价于`num - num/d*d`,所以会出现结果为负数的情况

##### 2.边界检查

当一个数字超过他的最大值，那么会溢出变为负数

所以要检查`0`,`负最小`,`正最大`

##### 3.包装类型

Integer源码分析

```java
    //装箱过程,推荐这个这种方式创建Integer对象
    public static Integer valueOf(int i) {
        
        if (i >= IntegerCache.low && i <= IntegerCache.high)//启用缓存
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }

    @Deprecated(since="9")//不推荐直接使用该构造函数来创建Integer对象
    public Integer(int value) {
        this.value = value;
    }

    /**
     * The value of the {@code Integer}.
     *
     * @serial
     */
    private final int value;

    //拆箱过程
    public int intValue() {
        return value;
    }

    private static class IntegerCache {
        static final int low = -128;//缓存下届
        static final int high;//缓存上届
        static final Integer cache[];//缓存数组

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];//分配内存
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);//缓存Integer

          	//缓存区间[-128,127]
            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

1.  拆箱装箱原理

```java
Integer i = 10;
//等价于
Integer i = Integer.valueOf(10);//装箱得到Integer类型值

int ii = i ;
//等价于
int ii = i.intValue();//拆箱得到int类型的值
```

  由上可知，当`i`为`null`时，拆箱失败，抛出`NullPointerException`所以包装类型拆箱之前要检查是否为`null`

2. `Integer`缓存机制

默认情况下`Integer`的会在`[-128,127]`之间缓存，刚好是一个`byte`的大小

对于==比较是否相等，对于引用类型比较是的引用，对于基本数据类型比较的是值

所以可以得出下结论

```java
Integer i1 = 10;
Integer i2 = 10;
i1 == i2;//true

Integer i3 = 128;
Integer i4 = 128;
i3 == i4;//false
```

而对于包装类型的 `>`  `<`会拆箱来比较值的大小

##### 4.随机数

产生随机数的两种方法:原理相同

```java
Random random = new Random();//当前时间为seed
random.nextInt();//包括其他基本数据类型

Math.random();//0.0<num<1.0,前时间为seed
```

> 种子不同，产生不同的随机数
>
> 种子相同，即使实例不同也产生相同的随机数

若非必要,`不要设置seed`

##### 5.静态变量的赋值

```java
class T{
    static{
        i = 100;
    }
    public static int i  = 10;

}
//T.i = 10;
```

> 静态变量在类初始化首先被加载的，JVM会去类中寻找所有的静态声明，然后分配内存空间，但还未赋值，之后JVM会根据静态赋值(静态成员和静态块)的先后顺序来执行。

所以为了防止歧义，要`先声明后使用`

##### 6.不要复写静态方法

> java中可以通过`Override`来增强或减弱父类的方法和行为，但`Override`是针对非静态方法，不能针对静态方法

`Base o = new Sub();`子类方法完全覆盖父类方法，访问权限可增大

定义:`Base`为表面类型，`Sub`为实际类型

对于非静态方法，`o`调用的是实际类型的方法

对于静态方法，`o`调用表面类型的方法

##### 7.构造函数

当子类没有显示调用父类的构造方法，会调用父类默认构造方法(没有参数)，若父类没有默认构造方法就会报错

并且在子类中，可以显示调用`super(args)`

```java
super(args1);
this(args2);//报错
```

`super`和`this`只有一个能处于构造方法的第一行

##### 8.静态内部类

静态内部类可以提高封装性

静态内部类与普通内部类的区别

> 1. 静态内部类不持有外部类的引用，普通内部类持有内部类引用
>
> 在普通内部类中可以直接访问外部类的属性、方法，即使是private修饰也可访问，而金泰内部类只能访问外部了的静态方法和静态属性，pirvate也可方访问
>
> 2. 通内部类与外部类之间是相互依赖的关系，内部类实例不能脱离外部类实例。而静态内部类可以独立存在
>
>    普通内部类不能声明static的方法和变量

##### 9.equals

使用`getClass()`来判断类型是否相同。因为`instanceof`可以判断继承关系，从而不满足`对称性`

```java
public boolean equals(Object obj){
  if(obj != null && obj,getClass() == this.getClass()){
    Person p = (Person)obj;
    if(p.gteName() == null || name == null){
      return false;
    }esle{
      return name.equalsIgnoreCase(p.getName());
    }
  }
  return false;
}
```

重写`equals()`必须重写`hasCode()`

##### 10.asList

基本数据类型数组转换为`List`

```java
int[] a = {1,2,3,4};
List list = Arrays.asList(a);//a[]整体被看作为一个参数
list.size();//1

Integer[] a = {1,2,3,4};
List list = Arrays.asList(a);
list.size();//a.length
```

转换为`list`之后，`list`是不可变

```java
private fianl E[] a;//传入的数组赋值它
//没有提供add insert delete方法
```

##### 11.泛型

1.  泛型的擦除

> Java泛型在编译期有效，在运行时间被删除，即泛型在编译后被擦除掉

List<String> List<Integer>  List<T>  ===> List

List<String>[]   ===> List[]

List<? extends E> List<? superE>  ===> List<E>

List<T extends Serializable && Cloneable> ===> List<Serializable>

2. 不能初始化泛型参数和数组

```java
T[] tt = new T[num];//不允许
List<T> list = new ArrayList<T>();//允许
```

3.上下界

上界可读`<? extends T>`

```java
public static void read(List<? extends E> list){
  for(E e:list){
    //...
  }
}
```

下界可写`<? super T>`

```java
public static void write(List<? super Number> list){
  list.add(1);
  list.add(1.1)
}
```

既可读又可写`<?>`

##### 12.synchronized

```java
//锁定obj对象
synchronized (obj){....}
//锁定this
public synchronized void method(){...}
//锁定Class对象
synchronized(MyClass){...}
//锁定当前的Class对象
public synchronized  static void method(){...}
```

##### 13.读取一个字符文件的行数

常规做法

```java
@Test
    public void test() throws IOException {
        File file = new File("test.txt");
        int count = 0;
        try(BufferedReader bufferedReader = new BufferedReader(new FileReader(file))){
            while (bufferedReader.readLine() != null){
                count++;
            }
            System.out.println("count: "+count);
        }
    }
```

利用Java API

```java
 @Test
    public void test() throws IOException {
        File file = new File("C:\\Users\\Young\\Desktop\\test.txt");
        try(LineNumberReader lineNumberReader = new LineNumberReader(new FileReader(file))){
            lineNumberReader.skip(file.length());
            //getLineNumber()返回从0~n-1
            int count = lineNumberReader.getLineNumber() + 1;
            System.out.println("count: "+count);
        }
    }
```

性能比较:第二种较好

##### 14.遍历Map

```java
@Test
    public void test() throws IOException {
        Map<String,Object> map = new HashMap<>(10);
        final int INDEX = 10;
        for(int i = 1; i < INDEX; i ++){
            map.put(i+"",i);
        }
        //遍历Map
        for(Map.Entry<String,Object> entry : map.entrySet()){
            System.out.println("k-v: "+entry.getKey() + ": " +entry.getValue());
        }
        //遍历key
        for (String k : map.keySet()){
            System.out.println("K: "+k);
        }
    }
```

##### 15.数组转List

```java
@Test
    public void test() throws IOException {
       String[] strArray = {"1","2","3"};
       List<String> strList = Arrays.asList(strArray);
       System.out.println(strList);
    }

//相反，list转数组
Object[] objects = list.toArray();
```

##### 16.将基本数据型、字符数组和对象转换为String类型

```java
String str  = String.valueOf(data)
```

##### 17.优先使用常量和确定值或使用Object.equals()来调用equals方法

```java
"abc".equals(str);

//Objects.equals(Object o1,Object o2);
public static boolean equals(Object a, Object b) {
        return (a == b) || (a != null && a.equals(b));
    }
```



