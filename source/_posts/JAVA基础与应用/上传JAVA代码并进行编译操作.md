---
title: 上传JAVA代码并进行编译操作
abbrlink: 3120348199
date: 2021-12-09 10:39:38
tags:
  - JAVA
categories: JAVA
description: 实现上传JAVA代码并进行编译返回
---
### 实现上传一段 JAVA代码  并进行执行  JAVA编译器操作

效果如下

![image-20211206214538310](https://qingyun-test.oss-cn-hangzhou.aliyuncs.com/img/image-20211206214538310.png?x-oss-process=style/qingyun)

## 具体实现

### 一：基于SpringBoot实现

#### 1 创建Springboot项目

pom文件

```xml
   <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

主要是这两个JAR包，提供web应用，因为和JAVA编译不存在太大关系，故不说明

#### 2 Controller层

```java
package edu.qingyun.controller;


import edu.qingyun.bytecode.InjectionSystem;
import edu.qingyun.maincomilerprocess.ClassInjector;
import edu.qingyun.maincomilerprocess.DynamicClassLoader;
import edu.qingyun.maincomilerprocess.DynamicLoaderEngine;
import org.springframework.web.bind.annotation.*;

import java.io.ByteArrayOutputStream;
import java.io.PrintStream;
import java.io.PrintWriter;

@RestController
@RequestMapping(value = "/compiler")
public class CompilerController {


    @PostMapping("/compilerTest")
    public String compiler(String code){
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        PrintWriter out = new PrintWriter(buffer, true);
        byte[] classBytes = DynamicLoaderEngine.compile(code, out, null);
        byte[] injectedClass = ClassInjector.injectSystem(classBytes);
        InjectionSystem.inject(null, new PrintStream(buffer, true), null);
        DynamicClassLoader classLoader = new DynamicClassLoader(this.getClass().getClassLoader());
        DynamicLoaderEngine.executeMain(classLoader, injectedClass, out);
        String s = buffer.toString();
        return s;
    }
}
```

Controller 也是主要逻辑实现类，基于此，我们开始往下探寻

```
ByteArrayOutputStream buffer = new ByteArrayOutputStream();
PrintWriter out = new PrintWriter(buffer, true);
```

这两句   `ByteArrayOutputStream `实现了一个输出流 ` 字节数组输出流在内存中创建一个字节数组缓冲区，所有发送到输出流的数据保存在该字节数组缓冲区中`

`PrintWriter`  在后续起到了输出错误信息的作用

```
byte[] classBytes = DynamicLoaderEngine.compile(code, out, null);
```

 `DynamicLoaderEngine.compile(code, out, null); ` 自己实现的主要编译类  。code 是需要编译的代码，out 是输出错误信息

##### DynamicLoaderEngine 类代码

```java
package edu.qingyun.maincomilerprocess;

import edu.qingyun.compiler.ClassCompiler;

import java.io.PrintWriter;
import java.util.List;



/**
 * 动态加载引擎，负责动态编译、加载类
 *
 *
 * @author xbc
 * @date 2019年1月9日
 *
 */
public class DynamicLoaderEngine {

    public static byte[] compile(String javaCode, PrintWriter out, List<String> options) {
        try {
            ClassCompiler classCompiler = new ClassCompiler();
            byte[] classBytes = classCompiler.compile(javaCode, out, options);
            if (null == classBytes) {
                return null;
            }
            return classBytes;
        }
        catch (Throwable t) {
            t.printStackTrace(out);
        }
        return null;
    }



    public static Class<?> loadClass(DynamicClassLoader classLoader, byte[] classBytes, PrintWriter out){
        try {
            Class<?> dynamicClass = classLoader.loadClassByBytes(classBytes);
            if (null == dynamicClass) {
                out.println("Failed to load class.");
                return null;
            }

            return dynamicClass;
        }
        catch (Throwable t) {
            t.printStackTrace(out);
        }
        return null;
    }

    /**
     * 加载类
     * 加载失败，则返回null, 同时out中包含错误信息
     *
     * @param classLoader 类加载器
     * @param javaCode 源码
     * @param out 错误信息输出
     * @param options 编译过程中的参数
     * @return
     */
    public static Class<?> loadClass(DynamicClassLoader classLoader, String javaCode, PrintWriter out, List<String> options) {
        try {
            byte[] classBytes = compile(javaCode, out, options);

            if (null == classBytes) {
                return null;
            }

            return loadClass(classLoader, classBytes, out);
        }
        catch (ClassFormatError e) {
            e.printStackTrace(out);
        }

        return null;
    }

    /**
     * 执行主类
     */
    public static boolean executeMain(DynamicClassLoader classLoader, String javaCode, PrintWriter out, List<String> options) {
        Class<?> loadedClass = loadClass(classLoader, javaCode, out, options);
        if (null == loadedClass) {
            return false;
        }

        return executeMain(loadedClass, out);
    }

    public static boolean executeMain(DynamicClassLoader classLoader, byte[] classBytes, PrintWriter out) {
        Class<?> loadClass = loadClass(classLoader, classBytes, out);
        if (loadClass == null) {
            return false;
        }

        return executeMain(loadClass, out);
    }

    public static boolean executeMain(Class<?> loadedClass, PrintWriter out) {
        ClassExecutor classExecutor =  new ClassExecutor();
        return classExecutor.executeMain(loadedClass, out);
    }
}
```



##### ClassCompiler

```java
package edu.qingyun.compiler;

import edu.qingyun.compiler.ClassFileManager;
import edu.qingyun.compiler.ClassJavaFileObject;
import edu.qingyun.compiler.SourceJavaFileObject;

import java.io.PrintWriter;
import java.util.ArrayList;
import java.util.List;

import javax.tools.Diagnostic;
import javax.tools.DiagnosticCollector;
import javax.tools.JavaCompiler;
import javax.tools.JavaFileObject;
import javax.tools.ToolProvider;

/**
 * 类编译器
 *
 * @author xbc
 * @date 2019年1月9日
 *
 */
public class ClassCompiler {

    /**
     * 编译源码，返回编译后的class字节数据
     * 返回null，表示编译失败
     * </br> 错误信息会放到out中。
     */
    public byte[] compile(String code, PrintWriter out, List<String> options) {
        if (null == out) {
            return null;
        }

        if (empty(code)) {
            out.print("Java code can't empty.");
            return null;
        }

        //获取类的全名称
        String fullClassName = getClassName(code);
        if (null == fullClassName) {
            out.print("The class full name can't be found from the code, for the java code format is not corrent.");
            return null;
        }

        if (null == options) {
            options = new ArrayList<String>();
        }

        try {
            //获取系统编译器
            JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();

            // 建立DiagnosticCollector对象, 用于搜集编辑期间的错误信息
            DiagnosticCollector<JavaFileObject> diagnostics = new DiagnosticCollector<JavaFileObject>();


            // 建立用于保存被编译文件名的对象
            // 每个文件被保存在一个从JavaFileObject继承的类中
            // *JavaFileManager就是tools包中使用的，管理java源文件和class类文件，抽象不同来源的这些数据的管理工具
            //  StandardJavaFileManager   compiler.getStandardFileManager返回类
            ClassFileManager fileManager = new ClassFileManager(compiler.getStandardFileManager(diagnostics, null, null));

            //設置需要被編譯的源碼
            List<JavaFileObject> jfiles = new ArrayList<JavaFileObject>();
            jfiles.add(new SourceJavaFileObject(fullClassName, code));

            JavaCompiler.CompilationTask task = compiler.getTask(out, fileManager, diagnostics, options, null, jfiles);

            // 编译源程序
            boolean success = task.call();
            if (success) {
                //如果编译成功,用类加载器加载该类
                ClassJavaFileObject classJavaFileObject = fileManager.getClassJavaFileObject();
                if (null == classJavaFileObject) {
                    out.println("Failed to compile class.");
                    outputErrorMsg(diagnostics, out);
                    return null;
                }

                return classJavaFileObject.getClassBytes();
            }
            else {
                outputErrorMsg(diagnostics, out);
                return null;
            }
        }
        catch (Throwable t) {
            t.printStackTrace(out);
        }

        return null;
    }

    private void outputErrorMsg(DiagnosticCollector<JavaFileObject> diagnostics, PrintWriter out) {
        for (Diagnostic<? extends JavaFileObject> diagnostic : diagnostics.getDiagnostics()) {
            outputErrorMsg(diagnostic, out);
        }
    }

    private void outputErrorMsg(Diagnostic<? extends JavaFileObject> diagnostic, PrintWriter out) {
        out.println("Code:[" + diagnostic.getCode() + "]");
        out.println("Kind:[" + diagnostic.getKind() + "]");
        out.println("Position:[" + diagnostic.getPosition() + "]");
        out.println("Start Position:[" + diagnostic.getStartPosition() + "]");
        out.println("End Position:[" + diagnostic.getEndPosition() + "]");
        out.println("Source:[" + diagnostic.getSource() + "]");
        out.println("Message:[" + diagnostic.getMessage(null) + "]");
        out.println("LineNumber:[" + diagnostic.getLineNumber() + "]");
        out.println("ColumnNumber:[" + diagnostic.getColumnNumber() + "]");
    }

    /**
     * 获取类的全名称
     * </br> 获取不到，返回null
     */
    private String getClassName(String code) {
        //get package
        String packageName = code.substring(code.indexOf("package") + 7, code.indexOf(";"));
        packageName = trim(packageName);

        //get class simple name
        String simpleName = code.substring(code.indexOf(" class ") + 7, code.indexOf("{"));
        simpleName = trim(simpleName);

        if (empty(simpleName)) {
            return null;
        }

        if (empty(packageName)) {
            return simpleName;
        }
        else {
            return packageName + "." + simpleName;
        }
    }

    private boolean empty(String str) {
        return null == str || str.isEmpty();
    }

    private String trim(String str) {
        if (null == str) {
            return str;
        }

        return str.trim();
    }
}

```

##### ClassFileManager

```java
package edu.qingyun.compiler;

import javax.tools.FileObject;
import javax.tools.ForwardingJavaFileManager;
import javax.tools.JavaFileObject;
import javax.tools.StandardJavaFileManager;
import java.io.IOException;

/**
 *JavaFileManager就是tools包中使用的，管理java源文件和class类文件，抽象不同来源的这些数据的管理工具
 * 类文件管理器 * 用于JavaCompiler将编译好后的class,保存到jclassObject中
 * @author qingyun
 * @date 2021/12/3
 */

public class ClassFileManager extends ForwardingJavaFileManager<StandardJavaFileManager> {

    private ClassJavaFileObject jclassObject;

    /**
     * Creates a new instance of ForwardingJavaFileManager.
     *
     * @param fileManager delegate to this file manager
     */
    protected ClassFileManager(StandardJavaFileManager fileManager) {
        super(fileManager);
    }

    /**
     * JavaCompiler在编译java文件的时候，生成的class二进制内容会放到这个JavaFileObject中
     */
    @Override
    public JavaFileObject getJavaFileForOutput(Location location, String className, JavaFileObject.Kind kind, FileObject sibling) throws IOException {
        if (jclassObject == null){
            jclassObject = new ClassJavaFileObject(className,kind);
        }
        return jclassObject;
    }

    public ClassJavaFileObject getClassJavaFileObject() {
        return jclassObject;
    }

}

```

##### ClassJavaFileObject

```java
package edu.qingyun.compiler;

import javax.tools.SimpleJavaFileObject;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.net.URI;

/**
 * class的文件JavaFileObject对象，用于存放JavaCompiler编译得到的Class二进制内容
 * </br>内容存在ByteArrayOutputStream中
 *
 * SimpleJavaFileObject是JavaFileObject接口的实现类，
 * 但是其中你可以发现很多的接口其实就是直接返回一个值，或者抛出一个异常，
 * 并且该类的构造器由protected修饰的，所以要实现复杂的功能，需要我们必须扩展这个类
 *
 * @author qingyun
 * @date 2021/12/3
 */
public class ClassJavaFileObject extends SimpleJavaFileObject {


    /**
     * ByteArrayOutputStream流是不需要关闭的
     */
    protected final ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();


    /**
     * Construct a SimpleJavaFileObject of the given kind and with the
     * given URI.
     *
     * @param name  the URI for this file object
     * @param kind the kind of this file object
     */
    public ClassJavaFileObject(String name, Kind kind) {
        super(URI.create("string:///" + name.replace('.', '/') + kind.extension), kind);
    }

    public byte[] getClassBytes(){
        return byteArrayOutputStream.toByteArray();
    }

    /**
     * 重写openOutputStream,将我们的输出流交给JavaCompiler,让它将编译好的Class装载进来
     */
    @Override
    public OutputStream openOutputStream() throws IOException {
        return byteArrayOutputStream;
    }
}
```

##### SourceJavaFileObject

```java
package edu.qingyun.compiler;

import javax.tools.SimpleJavaFileObject;
import java.net.URI;

/**
 * 存放java源码的JavaFileObject
 *
 *
 * @author xbc
 * @date 2019年1月9日
 *
 */
public class SourceJavaFileObject extends SimpleJavaFileObject {
    private String content;

    /**
     * 调用父类构造器,并设置content
     * @param className
     * @param content
     */
    public SourceJavaFileObject(String className, String content) { super(URI.create("string:///" + className.replace('.', '/') + Kind.SOURCE.extension), Kind.SOURCE);
        this.content = content;
    }

    /**
     * 实现getCharContent,使得JavaCompiler可以从content获取java源码
     */
    @Override
    public String getCharContent(boolean ignoreEncodingErrors) {
        return content;
    }
}
```

##### ClassModifier

```java
package edu.qingyun.bytecode;



/**
 * Class文件修改器
 * 参考：https://www.cnblogs.com/luobiao320/p/7651486.html
 */
public class ClassModifier {
    private static final int CONSTANT_POOL_COUNT_INDEX = 8;
    private static final int CONSTANT_Utf8_info = 1;
    private static final int[] CONSTANT_ITEM_LENGTH = {-1, -1, -1, 5, 5, 9, 9, 3, 3, 5, 5, 5, 5};
    private static final int u1 = 1;
    private static final int u2 = 2;
    private byte[] classByte;

    public ClassModifier(byte[] classByte) {
        this.classByte = classByte;
    }

    public byte[] modifyUTF8Constant4Class(Class<?> oldClass, Class<?> newClass) {
        return modifyUTF8Constant4ClassPath(oldClass.getName(), newClass.getName());
    }

    public byte[] modifyUTF8Constant4ClassPath(String oldClassName, String newClassName) {
        String oldReference = oldClassName.replace(".", "/");
        String newReference = newClassName.replace(".", "/");
        return modifyUTF8Constant4Reference(oldReference, newReference);
    }

    /**
     * 修改字符串常量池的符号引用
     */
    public byte[] modifyUTF8Constant4Reference(String oldReference, String newReference) {
        int cpc = getConstantPoolCount();
        int offset = CONSTANT_POOL_COUNT_INDEX + u2;
        for (int i = 0; i < cpc; i++) {
            int tag = ByteUtils.bytes2Int(classByte, offset, u1);
            if (tag == CONSTANT_Utf8_info) {
                int len = ByteUtils.bytes2Int(classByte, offset + u1, u2);
                offset += (u1 + u2);
                String str = ByteUtils.bytes2String(classByte, offset, len);
                if (str.equalsIgnoreCase(oldReference)) {
                    byte[] strBytes = ByteUtils.string2Bytes(newReference);
                    byte[] strLen = ByteUtils.int2Bytes(newReference.length(), u2);
                    classByte = ByteUtils.bytesReplace(classByte, offset - u2, u2, strLen);
                    //这里不只是替换，应该是填充，把新的字节数据插入到原来的位置，然后存在后面字节的向前或者先后移动
                    classByte = ByteUtils.bytesReplace(classByte, offset, len, strBytes);
                    return classByte;
                }
                else {
                    offset += len;
                }
            }
            else {
                offset += CONSTANT_ITEM_LENGTH[tag];
            }
        }

        //没有找到需要注入的引用字符串，直接返回原始数据
        return classByte;
    }

    public int getConstantPoolCount() {
        return ByteUtils.bytes2Int(classByte, CONSTANT_POOL_COUNT_INDEX, u2);
    }
}
```

##### ByteUtils

```java
package edu.qingyun.bytecode;

/**
 * 字节处理工具类
 * 参考：https://www.cnblogs.com/luobiao320/p/7651486.html
 */
public class ByteUtils {
    public static int bytes2Int(byte[] b, int start, int len) {
        int sum = 0;
        int end = start + len;
        for (int i = start; i < end; i++) {
            int n = ((int) b[i]) & 0xff;
            n <<= (--len) * 8;
            sum = n + sum;
        }
        return sum;
    }

    public static byte[] int2Bytes(int value, int len) {
        byte[] b = new byte[len];
        for (int i = 0; i < len; i++) {
            b[len - i - 1] = (byte) ((value >> 8 * i) & 0xff);
        }
        return b;
    }

    public static String bytes2String(byte[] b, int start, int len) {
        return new String(b, start, len);
    }

    public static byte[] string2Bytes(String str) {
        return str.getBytes();
    }

    public static byte[] bytesReplace(byte[] originalBytes, int offset, int len, byte[] replaceBytes) {
        byte[] newBytes = new byte[originalBytes.length + (replaceBytes.length - len)];
        System.arraycopy(originalBytes, 0, newBytes, 0, offset);
        System.arraycopy(replaceBytes, 0, newBytes, offset, replaceBytes.length);
        System.arraycopy(originalBytes, offset + len, newBytes, offset + replaceBytes.length, originalBytes.length - offset - len);
        return newBytes;
    }

}
```

##### InjectionSystem

```java
package edu.qingyun.bytecode;

import java.io.Console;
import java.io.IOException;
import java.io.InputStream;
import java.io.PrintStream;
import java.nio.channels.Channel;
import java.util.Properties;

/**
 * 自定义的可注入System，功能类同java.lang.System
 * </BR>
 * 该类的作用是，可以通过inject方法来注入System，即修改System的out、in和err的输出输入行为。修改之后的CustomSystem可以通过修改字节码的方式用于替代类原来的java.lang.System类
 * </BR>注入的时候，只是修改in out err的行为，其他System的方法都直接调用java.lang.System对应的方法（该类覆盖了所有java.lang.System的public方法）
 *
 * @author xbc
 * @date 2019年1月9日
 *
 */
public class InjectionSystem {
    public static InputStream in = System.in;

    public static PrintStream out = System.out;

    public static PrintStream err = System.err;

    private InjectionSystem() {
    }

    /**
     * 对System进行注入，即修改in out err三个参数的行为
     *
     *
     * @param iIn 不为null则修改in的行为
     * @param iOut 不为null则修改out的行为
     * @param iErr 不为null则修改err的行为
     */
    public static void inject(InputStream iIn, PrintStream iOut, PrintStream iErr) {
        if (null != in) {
            in = iIn;
        }

        if (null != iOut) {
            out = iOut;
        }

        if (null != iErr) {
            err = iErr;
        }
    }

    /**
     * 还原注入，即恢复为java.lang.System的功能
     */
    public static void restore() {
        in = System.in;
        out = System.out;
        err = System.err;
    }

    public static void setIn(InputStream in) {
        System.setIn(in);
    }

    public static void setOut(PrintStream out) {
        System.setOut(out);
    }

    public static void setErr(PrintStream err) {
        System.setErr(err);
    }

    public static Console console() {
        return System.console();
    }

    public static Channel inheritedChannel() throws IOException {
        return System.inheritedChannel();
    }

    public static void setSecurityManager(final SecurityManager s) {
        System.setSecurityManager(s);
    }

    public static SecurityManager getSecurityManager() {
        return System.getSecurityManager();
    }

    public static long currentTimeMillis() {
        return System.currentTimeMillis();
    }

    public static long nanoTime() {
        return System.nanoTime();
    }

    public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length) {
        System.arraycopy(src, srcPos, dest, destPos, length);
    }

    public static int identityHashCode(Object x) {
        return System.identityHashCode(x);
    }

    public static Properties getProperties() {
        return System.getProperties();
    }

    public static void setProperties(Properties props) {
        System.setProperties(props);
    }

    public static String getProperty(String key) {
        return System.getProperty(key);
    }

    public static String getProperty(String key, String def) {
        return System.getProperty(key, def);
    }

    public static String setProperty(String key, String value) {
        return System.setProperty(key, value);
    }

    public static String clearProperty(String key) {
        return System.clearProperty(key);
    }

    public static String getenv(String name) {
        return System.getenv(name);
    }

    public static java.util.Map<String, String> getenv() {
        return System.getenv();
    }

    public static void exit(int status) {
        System.exit(status);
    }

    public static void gc() {
        System.gc();
    }

    public static void runFinalization() {
        System.runFinalization();
    }



    public static void load(String filename) {
        System.load(filename);
    }

    public static void loadLibrary(String libname) {
        System.loadLibrary(libname);
    }
}
```

##### ClassExecutor

```java
package edu.qingyun.maincomilerprocess;

import java.io.PrintWriter;
import java.lang.reflect.Method;

/**
 * Class执行器
 * @author qingyun
 * @date 2021/12/3
 */
public class ClassExecutor {
    public boolean executeMain(Class<?> cls , PrintWriter out){
        Method method;
        try{
            method = cls.getMethod("main", new Class[]{String[].class});
            method.invoke(null,new String[]{null});
        }catch (Throwable t){
            t.printStackTrace();
        }
        return false;
    }
}
```

##### ClassInjector

```java
package edu.qingyun.maincomilerprocess;


import edu.qingyun.bytecode.ClassModifier;
import edu.qingyun.bytecode.InjectionSystem;

/**
 *
 * 对类进行注入操作，即修改Class二进制中的字符串常量的符号引用，从而达到替换功能的目的
 *
 * @author qingyun
 * @date 2021/12/3
 */
public class ClassInjector {
    public static byte[] injectSystem(byte[] classByte){
        return inject(classByte,System.class, InjectionSystem.class);
    }

    /**
     * 注入Class类，修改其相应类的符号引用
     * </br>返回null，表示修改失败或者没有找到待修改的内容
     *
     *
     * @param classBytes 待修改的class内容
     * @param oldClassStrRef 老的类的符号引用
     * @param newClassStrRef 新的类的符号引用
     * @return 修改之后的class文件内容byte[]
     */
    public static byte[] inject(byte[] classBytes, String oldClassStrRef, String newClassStrRef) {
        ClassModifier classModifier = new ClassModifier(classBytes);
        return classModifier.modifyUTF8Constant4Reference(oldClassStrRef, newClassStrRef);
    }

    public static byte[] inject(byte[] classBytes, Class<?> oldClass, Class<?> newClass) {
        ClassModifier classModifier = new ClassModifier(classBytes);
        return classModifier.modifyUTF8Constant4Class(oldClass, newClass);
    }
}
```

##### DynamicClassLoader

```java
package edu.qingyun.maincomilerprocess;

import java.net.URL;
import java.net.URLClassLoader;

public class DynamicClassLoader extends URLClassLoader {
    public DynamicClassLoader(ClassLoader parent) {
        super(new URL[0], parent);
    }

    public Class<?> findClassByClassName(String className) throws ClassNotFoundException {
        return this.findClass(className);
    }

    public Class<?> loadClassByBytes(byte[] classData) {
        return this.defineClass(null, classData, 0, classData.length);
    }
}
```

##### DynamicExecuteService（独立的实现）

一个Service类

```java
package edu.qingyun.server;


import edu.qingyun.bytecode.InjectionSystem;
import edu.qingyun.maincomilerprocess.ClassInjector;
import edu.qingyun.maincomilerprocess.DynamicClassLoader;
import edu.qingyun.maincomilerprocess.DynamicLoaderEngine;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.PrintStream;
import java.io.PrintWriter;
import java.net.URL;
import java.net.URLClassLoader;
import java.util.ArrayList;
import java.util.List;

/**
 * 动态执行服务
 *
 * @author xbc
 * @date 2019年1月10日
 *
 */
public class DynamicExecuteService {
    private URLClassLoader pClassLoader;

    /**
     * 构造函数
     *
     * @param pClassLoader 父类加载器
     */
    public DynamicExecuteService(URLClassLoader pClassLoader) {
        this.pClassLoader = pClassLoader;
    }

    /**
     * 动态执行代码</BR>
     * 1、使用自定义类加载器加载类</BR>
     * 2、修改类中的java.lang.System类为InjectionSystem</BR>
     * 3、将InjectionSystem的out输出到ByteArrayOutputStream</BR>
     * 4、执行过程中的所有错误都整理成字符串返回</BR>
     * 5、类中的执行结果也通过字符串返回</BR>
     * 6、4和5的输出结果不会同时存在</BR>
     *
     * @param code 源码
     * @return 返回错误或者类的执行输出
     */
    public String executeDynamically(String code) {
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        PrintWriter out = new PrintWriter(buffer, true);

        try {
            String path = "";
            path = getClassPath(path);

            List<String> options = new ArrayList<String>();
            options.add("-classpath");
            options.add(path);
            options.add("-encoding");
            options.add("UTF-8");

            byte[] classBytes = DynamicLoaderEngine.compile(code, out, options);
            byte[] injectedClass = ClassInjector.injectSystem(classBytes);

            InjectionSystem.inject(null, new PrintStream(buffer, true), null);

            DynamicClassLoader dynamicClassLoader = new DynamicClassLoader(pClassLoader);
            DynamicLoaderEngine.executeMain(dynamicClassLoader, injectedClass, out);

            InjectionSystem.restore();
        }
        catch (Throwable t) {
            t.printStackTrace(out);
        }
        finally {
            out.close();
        }

        return buffer.toString();
    }

    private String getClassPath(String path) {

        URLClassLoader classLoader = pClassLoader;
        while (true) {
            for (URL url : classLoader.getURLs()) {
                path = path + url.getFile() + File.pathSeparator;
            }

            if (null == classLoader.getParent()) {
                break;
            }

            if (classLoader.getParent() instanceof URLClassLoader) {
                classLoader = (URLClassLoader) classLoader.getParent();
            }
            else {
                break;
            }
        }

        return path;
    }
}
```