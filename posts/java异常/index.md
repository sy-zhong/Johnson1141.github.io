# Java异常与错误机制

Java异常与错误机制

## 引入  
程序运行时随时会发生异常，有些是可预料到的，比如获取列表中第二个值，可能整个列表长度都没有2，有些是运行过程中随时随地会出现的，比如oom，StackOverflow，泛型赋值类型不匹配，我们只能尽量减缓其出现，但无法保证不出现，所以无需捕获也没有一个特定的地方捕获（除非你明确知道这里会出现运行时异常，并且代码无法优化，example?）。  
  
引入java异常机制，通常是为了处理程序在编译和运行时出现意料之外的事情，vm通知你在程序中犯错误了，让你有机会去处理这个错误。  
  
## throwable  
java异常体系顶层是throwable  
有exception和error两类异常。  
error 一般表示java运行过程中jvm出现的问题，通常有oom，stackoverflow，一般情况下，这种错误是不应该被应用程序处理的（比如oom）  
exception 表示程序能捕获且进行处理的异常，又分为运行时异常和受检异常（编译时异常）  
运行时异常（runtimeException类及其子类）：npe，越界 and so on  
受检异常（除了runtimeException以外的异常）  
  
特别的，程序能捕获throwable，也就意味着能同时捕获 exception和error  
但是很多时候捕获error是没有意义的，比如oom，如果是代码块中申请了一块大空间导致的oom，可能有效  
但是假设本身memory已经岌岌可危了，极有可能在catch代码块中又抛出oom。  
  
  
## 源码解析  
java抛异常为什么是个昂贵的操作，抛出异常时做了什么？ 复写方法以及建议  
```java
// exception 
    public Exception() {
        super();
    }
// end

// throw 
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
查看以上的接口，可以看到构造一个异常的时候，大部分构造方法都会调用 fillInStackTrace 这个native方法， 这个本地方法做的就是 填充执行堆栈跟踪，这个方法有些耗时，导致异常的构造会是个重的操作。
但也可以看到 ```writableStackTrace``` 这个参数可以用来手动控制是否爬栈，在仅是记录业务异常时可以使用。
```enableSuppression`` 关键字在try-with-resouces部分讨论
  
## 常见关键字  
### try,catch,fianlly  
try, catch 不用多说
这里主要讨论的是finally的作用。
首先查看一个常见的代码
```
try{
    dosomething();
}catch(Exception e){

}

```

综合上述：
* finally中可以修改对象值，但是注意的是基本数据类型和引用类型的异同
* 若是try和finally中都存在return 会return finally中的值
* 不建议在finally中返回值




### throw,throws  
  
## 最佳实践  
### 如何自定义异常  
Q:为什么要自定义异常，如果只是要报错信息，直接抛出errorMsg和堆栈不行吗？  
GPT回答（觉得话术还可以）：自定义异常可以让代码更加清晰易懂，同时也可以提高代码的可重用性和可维护性。自定义异常可以根据业务需求定义不同的异常类型，并且可以在异常中添加一些额外的信息，比如异常发生的时间、异常发生的位置等。这些信息可以帮助开发人员更快地定位和解决问题。另外，自定义异常还可以让代码更加规范化，提高代码的可读性和可维护性，减少代码的重复性。因此，建议在需要处理异常的情况下，使用自定义异常来提高代码的质量。  
  
### 异常后的操作  
  
### 常见异常以及处理方式  
  
### 案例  
  
#### case1  
  
  
## 常见框架的异常处理  
### 线程池  
  
### AspectJ  
after  
  
### 待补充
