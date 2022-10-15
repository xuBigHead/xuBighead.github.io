# Throwable

Throwable是所有异常的父类。

![img](../../Image/2022/07/220715-3.png)



在 Java 中，所有的异常都有一个共同的祖先 `java.lang` 包中的 `Throwable` 类。`Throwable` 类有两个重要的子类 `Exception`（异常）和 `Error`（错误）。`Exception` 能被程序本身处理(`try-catch`)， `Error` 是无法处理的(只能尽量避免)。

`Exception` 和 `Error` 二者都是 Java 异常处理的重要子类，各自都包含大量子类。

- **`Exception`** :程序本身可以处理的异常，可以通过 `catch` 来进行捕获。`Exception` 又可以分为 受检查异常(必须处理) 和 不受检查异常(可以不处理)。
- **`Error`** ：`Error` 属于程序无法处理的错误 ，没办法通过 `catch` 来进行捕获 。例如，Java 虚拟机运行错误（`Virtual MachineError`）、虚拟机内存不够错误(`OutOfMemoryError`)、类定义错误（`NoClassDefFoundError`）等 。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。



## 常用方法

- **`public string getMessage()`**:返回异常发生时的简要描述
- **`public string toString()`**:返回异常发生时的详细信息
- **`public string getLocalizedMessage()`**:返回异常对象的本地化信息。使用 `Throwable` 的子类覆盖这个方法，可以生成本地化信息。如果子类没有覆盖该方法，则该方法返回的信息与 `getMessage（）`返回的结果相同
- **`public void printStackTrace()`**:在控制台上打印 `Throwable` 对象封装的异常信息



## 源码解析

### 构造方法

> java.lang.Throwable
```java
    public Throwable() {
        fillInStackTrace();
    }
    public Throwable(String message) {
        fillInStackTrace();
        detailMessage = message;
    }
    public Throwable(String message, Throwable cause) {
        fillInStackTrace();
        detailMessage = message;
        this.cause = cause;
    }
    public Throwable(Throwable cause) {
        fillInStackTrace();
        detailMessage = (cause==null ? null : cause.toString());
        this.cause = cause;
    }
    protected Throwable(String message, Throwable cause,
                        boolean enableSuppression,
                        boolean writableStackTrace) {
        if (writableStackTrace) {
            fillInStackTrace();
        } else {
            stackTrace = null;
        }
        detailMessage = message;
        this.cause = cause;
        if (!enableSuppression)
            suppressedExceptions = null;
    }
```

> java.lang.Throwable.fillInStackTrace
```java
public synchronized Throwable fillInStackTrace() {
    if (stackTrace != null ||
        backtrace != null /* Out of protocol state */ ) {
        fillInStackTrace(0);
        stackTrace = UNASSIGNED_STACK;
    }
    return this;
}

private native Throwable fillInStackTrace(int dummy);
```



# Error

错误：JVM内部的严重问题。无法恢复。程序人员不用处理。 

## 源码解析
> 构造器
```java

```

# Exception
普通的问题。通过合理的处理，程序还可以回到正常执行流程。要求编程人员要进行处理。

## 非受检异常
RuntimeException及其子类异常也叫非受检异常(unchecked exception)，这类异常是编程人员的逻辑问题，Java编译器不进行强制要求处理。

Java 代码在编译过程中 ，我们即使不处理不受检查异常也可以正常通过编译。

`RuntimeException` 及其子类都统称为非受检查异常，例如：`NullPointerException`、`NumberFormatException`（字符串转换为数字）、`ArrayIndexOutOfBoundsException`（数组越界）、`ClassCastException`（类型转换错误）、`ArithmeticException`（算术错误）等。



### 源码解析
## 受检异常
非RuntimeException也叫受检异常(checked exception)，这类异常是由一些外部的偶然因素所引起的。Java编译器强制要求处理。

Java 代码在编译过程中，如果受检查异常没有被 `catch`/`throw` 处理的话，就没办法通过编译 。比如下面这段 IO 操作的代码。

除了`RuntimeException`及其子类以外，其他的`Exception`类及其子类都属于检查异常 。常见的受检查异常有： IO 相关的异常、`ClassNotFoundException` 、`SQLException`...。



### 源码解析
## 源码解析  



# 异常关键字

## try-catch-finally

### try

 用于捕获异常。其后可接零个或多个 `catch` 块，如果没有 `catch` 块，则必须跟一个 `finally` 块。



### catch

用于处理 try 捕获到的异常。

多个catch块时候，最多只会匹配其中一个异常类且只会执行该catch块代码，而不会再执行其它的catch块;

匹配catch语句的顺序为从上到下，也可能所有的catch都没执行。因此要先catch子类异常再catch父类异常。



### finally

无论是否捕获或处理异常，`finally` 块里的语句都会被执行。当在 `try` 块或 `catch` 块中遇到 `return` 语句时，`finally` 语句块将在方法返回之前被执行。

- try、catch、finally三个语句块均不能单独使用，三者可以组成 try-catch-finally、try-catch、try-finally三种结构;
- catch语句可以有一个或多个，finally语句最多一个;
- try、catch、finally三个代码块中变量的作用域为代码块内部，分别独立而不能相互访问。



**在以下 3 种特殊情况下，`finally` 块不会被执行：**

1. 在 `try` 或 `finally `块中用了 `System.exit(int)`退出程序。但是，如果 `System.exit(int)` 在异常语句之后，`finally` 还是会被执行
2. 程序所在的线程死亡。
3. 关闭 CPU。



### 返回值优先级

当 try 语句和 finally 语句中都有 return 语句时，在方法返回之前，finally 语句的内容将被执行，并且 finally 语句的返回值将会覆盖原始的返回值。

```java
public static int f(int value) {
    try {
        return value * value;
    } finally {
        if (value == 2) {
          	// if结果为true时会返回finally块里的返回值
            return 0;
        }
    }
}
```



### 代码示例

```java
public void exception() {
    try {
        String errorJsonString = "{";
        JSON.parse(errorJsonString);
    } catch (JSONException e) {
        // 捕获解析json字符串的异常
        log.error("捕获json解析异常: {}", e.getMessage());
    } catch (Exception e) {
        // catch可以有多个，finally最多只能有一个
        log.error("捕获未知异常: {}", e.getMessage());
    } finally {
        log.info("一定会执行的功能");
    }
}
```



## throw

- throw关键字是用于方法体内部，用来抛出一个Throwable类型的异常。
- 如果抛出了检查异常，则还应该在方法头部声明方法可能抛出的异常类型。该方法的调用者也必须检查处理抛出的异常。如果所有方法都层层上抛获取的异常，最终JVM会进行处理，处理也很简单，就是打印异常消息和堆栈信息。

## throws
- 方法中存在受检异常，如果不对其捕获，则必须在方法头中显式声明该异常，告知方法调用者此方法有异常，需要进行处理。 
- 方法中存在异常，方法头中使用关键字throws，后面接上要声明的异常。若声明多个异常，则使用逗号分割。
- 父类的方法没有声明异常，则子类继承方法后，也不能声明异常。

```java
public void checkException() throws Exception {
    int randomNumber = new Random().nextInt(10);
    if ((randomNumber & 1) == 1) {
        // 受检异常要么用try-catch进行处理，要么用throws抛出，交给调用者处理
        throw new Exception("受检异常");
    }
    try {
        throw new IOException("IOException也是受检异常");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
public void unCheckException() {
    // 非受检异常可以不用处理，直接抛出
    throw new RuntimeException("非受检异常");
}
```
