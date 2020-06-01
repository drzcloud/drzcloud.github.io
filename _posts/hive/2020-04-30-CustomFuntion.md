## 1 自定义函数

**1）Hive 自带了一些函数，比如：max/min等，但是数量有限，自己可以通过自定义UDF来方便的扩展。**

**2）当Hive提供的内置函数无法满足你的业务处理需要时，此时就可以考虑使用用户自定义函数（UDF：user-defined function）。**

**3）根据用户自定义函数类别分为以下三种：**

​	**（1）UDF（User-Defined-Function） 一进一出**

​	**（2）UDAF（User-Defined Aggregation Function） 聚集函数，多进一出 类似于：count/max/min**

​	**（3）UDTF（User-Defined Table-Generating Functions）一进多出，如lateral view explode()**

**4）[官方文档地址](https://cwiki.apache.org/confluence/display/Hive/HivePlugins)**

**5）编程步骤：**

​	**（1）继承Hive提供的类**

```java
org.apache.hadoop.hive.ql.udf.generic.GenericUDF  
org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
```

​	**（2）实现类中的抽象方法**

​	**（3）在hive的命令行窗口创建函数**

​		**添加jar**

```
add jar linux_jar_path
```

​		**创建function**

```sql
create [temporary] function [dbname.]function_name AS class_name;
```

​	**（4）在hive的命令行窗口删除函数**

```sql
drop [temporary] function [if exists] [dbname.]function_name;
```

## 2 自定义UDF函数

**0）需求:**

自定义一个UDF实现计算给定字符串的长度，例如：

```sql
hive(default)> select my_len("abcd");
4
```

**1）创建一个Maven工程Hive**

**2）导入依赖**

```xml
<dependencies>
		<dependency>
			<groupId>org.apache.hive</groupId>
			<artifactId>hive-exec</artifactId>
			<version>3.1.2</version>
		</dependency>
</dependencies>
```

**3）创建一个类**

```java
package com.iisrun.hive;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentLengthException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentTypeException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;

/**
 * 需求: 计算传入的字符串的长度并返回
 *
 * 自定义UDF ,需要继承GenericUDF类
 *
 */
public class MyUDF extends GenericUDF {

    /**
     * 对输入参数的判断处理和返回值类型的一个约定。
     * @param arguments  传入到函数的参数的类型对应的ObjectInspector
     * @return
     * @throws UDFArgumentException
     */
    @Override
    public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {
        if(arguments == null || arguments.length !=1){
            throw new UDFArgumentLengthException("Input Args Length Error !!!!");
        }
        if(!arguments[0].getCategory().equals(ObjectInspector.Category.PRIMITIVE)){
            throw new UDFArgumentTypeException(0,"Input Args Type Error !!!!");
        }
        //约定函数的返回值类型为int
        return PrimitiveObjectInspectorFactory.javaIntObjectInspector ;
    }

    /**
     * 函数的逻辑处理
     * @param arguments  传入到函数的参数
     * @return
     * @throws HiveException
     */
    @Override
    public Object evaluate(DeferredObject[] arguments) throws HiveException {
        //取出参数
        Object o = arguments[0].get();
        if(o == null){
            return 0 ;
        }
        return o.toString().length();
    }

    @Override
    public String getDisplayString(String[] children) {
        return "";
    }
}
```

 **4）打成jar包上传到服务器/opt/module/hive/datas/myudf.jar**

**5）将jar包添加到hive的classpath**

```java
hive (default)> add jar /opt/module/hive/datas/myudf.jar;
```

**6）创建临时函数与开发好的java class关联**

```java
hive (default)> create temporary function my_len as "com.iisrun.hive.MyStringLength";
```

**7）即可在hql中使用自定义的函数strip** 

```sql
hive (default)> select ename,my_len(ename) ename_len from emp;
```

## 3 自定义UDTF函数1

**0）需求** 

​	自定义一个UDTF实现将一个任意分割符的字符串切割成独立的单词，例如：

```sql
hive(default)> select myudtf("hello,world,hadoop,hive", ",");

hello
world
hadoop
hive
```

**1）代码实现**

```java
package com.iisrun.hive;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;

import java.util.ArrayList;
import java.util.List;

/**
 * 将指定的字符串通过指定的分隔符，进行拆分，返回多行数据
 *
 * 自定义UDTF,需要继承GenericUDTF
 */
public class MyUDTF extends GenericUDTF {

    private List<String> outList = new ArrayList<>();

    /**
     * 约定函数输出的列的名字  和 列的类型。
     * @param argOIs
     * @return
     * @throws UDFArgumentException
     */
    @Override
    public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {
        //约定函数输出的列的名字
        ArrayList<String>  fieldNames  = new ArrayList<>() ;
        // 约定函数输出的列的类型
        ArrayList<ObjectInspector> fieldOIs = new ArrayList<>();

        fieldNames.add("word");
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);

        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames,fieldOIs);
    }

    /**
     * 函数的处理逻辑
     * @param args
     * @throws HiveException
     */
    @Override
    public void process(Object[] args) throws HiveException {
        //简单判断
        if(args==null || args.length <2){
            throw new HiveException("Input Args Length Error");
        }
        //获取待处理的数据
        String argsData = args[0].toString();
        //获取分隔符
        String argsSplit = args[1].toString();

        //切割数据
        String[] words = argsData.split(argsSplit);

        //迭代写出
        for (String word : words) {
            //因为集合是复用的， 使用前先清空之前数据
            outList.clear();
            //写出
            outList.add(word);
            forward(outList);
        }
    }

    @Override
    public void close() throws HiveException {
        
    }
}
```

**2）打成jar包上传到服务器/opt/module/hive/data/myudtf.jar**

**3）将jar包添加到hive的classpath下**

```java
hive (default)> add jar /opt/module/hive/data/myudtf.jar;
```

**4）创建临时函数与开发好的java class关联**

```
hive (default)> create temporary function myudtf as "com.iisrun.hive.MyUDTF";
```

**5）使用自定义的函数**

```java
hive (default)> select myudtf("hello,world,hadoop,hive",",") ;
```

## 3 自定义UDTF函数2

**0）需求** 

​	将出入的字符串，按照分隔符拆分，输出多行多列，例如：

```
输入数据: "hello_world,hadoop_hive,java_php"
结果:     hello   world
         hadoop  hive
         java    php
```

**1）代码实现**

```java
package com.iisrun.hive;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;

import java.util.ArrayList;
import java.util.List;

/**
 * 需求:
 *    将出入的字符串，按照分隔符拆分，输出多行多列
 *
 *    输入数据: "hello_world,hadoop_hive,java_php"
 *
 *    结果:     hello   world
 *             hadoop  hive
 *             java    php
 *
 *    函数的调用: my_explode2("hello_world,hadoop_hive,java_php",",","_");
 */
public class MyUDTF2 extends GenericUDTF {

    private List<String> outList = new ArrayList<>();

    /**
     * 约定函数输出的列的名字  和 列的类型。
     * @param argOIs
     * @return
     * @throws UDFArgumentException
     */
    @Override
    public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {
        //约定函数输出的列的名字
        ArrayList<String>  fieldNames  = new ArrayList<>() ;
        // 约定函数输出的列的类型
        ArrayList<ObjectInspector> fieldOIs = new ArrayList<>();

        fieldNames.add("word1");
        fieldNames.add("word2");
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);

        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames,fieldOIs);
    }

    /**
     * 函数的处理逻辑
     * @param args
     * @throws HiveException
     *
     * my_explode2("hello_world,hadoop_hive,java_php",",","_");
     */
    @Override
    public void process(Object[] args) throws HiveException {
        //简单判断
        if(args==null || args.length <3){
            throw new HiveException("Input Args Length Error");
        }
        //获取待处理的数据
        String argsData = args[0].toString();
        //获取分隔符1
        String argsSplit1 = args[1].toString();  // ","
        //获取分隔符2
        String argsSplit2 = args[2].toString();  // "_"

        //切割数据
        String[] words = argsData.split(argsSplit1);
        //迭代写出
        for (String word : words) {
            //清空集合
            outList.clear();
            // hello_world
            //继续切割
            String[] currentWords = word.split(argsSplit2);
            for (String currentWord : currentWords) {
                outList.add(currentWord);
            }
            //写出
            forward(outList);
        }
    }

    @Override
    public void close() throws HiveException {

    }
}
```

**2）打成jar包上传到服务器/opt/module/hive/data/myudtf2.jar**

**3）将jar包添加到hive的classpath下**

```java
hive (default)> add jar /opt/module/hive/data/myudtf2.jar;
```

**4）创建临时函数与开发好的java class关联**

```java
hive (default)> create temporary function myudtf2 as "com.iisrun.hive.MyUDTF2";
```

**5）使用自定义的函数**

```sql
hive (default)> select myudtf2("hello_world,hadoop_hive,java_php",",") ;
```