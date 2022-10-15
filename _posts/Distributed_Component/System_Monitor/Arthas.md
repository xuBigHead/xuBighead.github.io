# Arthas
Arthas 是Alibaba开源的Java诊断工具，采用命令行交互模式，提供了丰富的功能，是排查jvm相关问题的利器。
下面是它能做的一些事情：

- 提供性能看板，包括线程、cpu、内存等信息，并且会定时的刷新。
- 根据各种条件查看线程快照。比如找出cpu占用率最高的n个线程等
- 输出jvm的各种信息，如gc算法、jdk版本、ClassPath等
- 查看/设置sysprop和sysenv
- 查看某个类的静态属性，也可以通过ognl语法执行一些语句
- 查看已加载的类的详细信息，比如这个类从哪个jar包加载的。也可以查看类的方法的信息
- dump某个类的字节码到指定目录
- 直接反编译指定的类
- 查看类加载器的一些信息
- 可以让jvm重新加载某个类
- 监控方法的执行，同时可以获取到执行的入参、出参以及抛出的异常
- 追踪方法执行的调用栈，以及各个方法的调用时间

## 参考资料


# 运行原理

![1](../../../Image/2022/03/220318-1.png)





# 安装和启动

　下载`arthas-boot.jar`，再用`java -jar`命令启动：



## 下载安装

```
[root@izuf6e56zt7ebjhby2hxboz ~]# wget https://arthas.aliyun.com/arthas-boot.jar
--2022-03-17 17:39:11--  https://arthas.aliyun.com/arthas-boot.jar
Resolving arthas.aliyun.com (arthas.aliyun.com)... 203.119.211.244
Connecting to arthas.aliyun.com (arthas.aliyun.com)|203.119.211.244|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 141891 (139K) [application/java-archive]
Saving to: ‘arthas-boot.jar’

100%[========================================================================================================================>] 141,891  

2022-03-17 17:39:11 (386 KB/s) - ‘arthas-boot.jar’ saved [141891/141891]
```



`arthas-boot`是`Arthas`的启动程序，它启动后，会列出所有的Java进程，用户可以选择需要诊断的目标进程。



## 启动

```
[root@izuf6e56zt7ebjhby2hxboz ~]# java -jar arthas-boot.jar 
[INFO] arthas-boot version: 3.5.5
[INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
* [1]: 31358 tool-1.0-SNAPSHOT.jar
```



选择需要监控进程前的序号，输入 `1` ，再Enter 回车，便会打印 arthas 的banner。



```
[INFO] Start download arthas from remote server: https://arthas.aliyun.com/download/3.5.6?mirror=aliyun
[INFO] Download arthas success.
[INFO] arthas home: /root/.arthas/lib/3.5.6/arthas
[INFO] Try to attach process 31358
[INFO] Attach process 31358 success.
[INFO] arthas-client connect 127.0.0.1 3658
  ,---.  ,------. ,--------.,--.  ,--.  ,---.   ,---.                           
 /  O  \ |  .--. ''--.  .--'|  '--'  | /  O  \ '   .-'                          
|  .-.  ||  '--'.'   |  |   |  .--.  ||  .-.  |`.  `-.                          
|  | |  ||  |\  \    |  |   |  |  |  ||  | |  |.-'    |                         
`--' `--'`--' '--'   `--'   `--'  `--'`--' `--'`-----'                          

wiki       https://arthas.aliyun.com/doc                                        
tutorials  https://arthas.aliyun.com/doc/arthas-tutorials.html                  
version    3.5.6                                                                
main_class                                                                      
pid        31358                                                                
time       2022-03-17 17:39:29   
```



## 卸载

```
 [root@izuf6e56zt7ebjhby2hxboz ~]# rm -rf ~/.arthas/
```



# 语法命令

![1](../../../Image/2022/03/220318-2.png)



## 基础

### help

```
[arthas@31358]$ help
```



`help` 可以查看 arthas 的所有命令

```
NAME                DESCRIPTION
help                Display Arthas Help
auth                Authenticates the current session
keymap              Display all the available keymap for the specified connection.
sc                  Search all the classes loaded by JVM
sm                  Search the method of classes loaded by JVM
classloader         Show classloader info
jad                 Decompile class
getstatic           Show the static field of a class
monitor             Monitor method execution statistics, e.g. total/success/failure count, average rt, fail rate, etc.
stack               Display the stack trace for the specified class and method
thread              Display thread info, thread stack
trace               Trace the execution time of specified method invocation.
watch               Display the input/output parameter, return object, and thrown exception of specified method invocation
tt                  Time Tunnel
jvm                 Display the target JVM information                                           
memory              Display jvm memory info.                                                     
perfcounter         Display the perf counter information.                                        
ognl                Execute ognl expression.                                                     
mc                  Memory compiler, compiles java files into bytecode and class files in memory.
redefine            Redefine classes. @see Instrumentation#redefineClasses(ClassDefinition...)   
retransform         Retransform classes. @see Instrumentation#retransformClasses(Class...)       
dashboard           Overview of target jvm's thread, memory, gc, vm, tomcat info.                
dump                Dump class byte array from JVM                                               
heapdump            Heap dump                                                                    
options             View and change various Arthas options                                       
cls                 Clear the screen                                                             
reset               Reset all the enhanced classes                                               
version             Display Arthas version                                                       
session             Display current session information                                          
sysprop             Display, and change the system properties.                                   
sysenv              Display the system env.                                                      
vmoption            Display, and update the vm diagnostic options.                               
logger              Print logger info, and update the logger level                               
history             Display command history                                                      
cat                 Concatenate and print files                                                  
base64              Encode and decode using Base64 representation                                
echo                write arguments to the standard output                                       
pwd                 Return working directory name                                                
mbean               Display the mbean information                                                
grep                grep command for pipes.                                                      
tee                 tee command for pipes.                                                       
profiler            Async Profiler. https://github.com/jvm-profiling-tools/async-profiler        
vmtool              jvm tool                                                                     
stop                Stop/Shutdown Arthas server and exit the console. 
```



### cls   

### session

### reset

### version

### quit

exit或者quit只会退出交互界面，不会关闭attach的arthas进程。



### stop 

用 `exit` 或者 `quit` 或者 `stop` 命令可以退出Arthas。



### shutdown

关闭Arthas。



### history

### options

打开或关闭某些功能。



| 名称               | 默认值 | 描述                                                         |
| ------------------ | ------ | ------------------------------------------------------------ |
| unsafe             | false  | 是否支持对系统级别的类进行增强，打开该开关可能导致把JVM搞挂，请慎重选择！ |
| dump               | false  | 是否支持被增强了的类dump到外部文件中，如果打开开关，class文件会被dump到`/${application dir}/arthas-class-dump/`目录下，具体位置详见控制台输出 |
| batch-re-transform | true   | 是否支持批量对匹配到的类执行retransform操作                  |
| json-format        | false  | 是否支持json化的输出                                         |
| disable-sub-class  | false  | 是否禁用子类匹配，默认在匹配目标类的时候会默认匹配到其子类，如果想精确匹配，可以关闭此开关 |
| debug-for-asm      | false  | 打印ASM相关的调试信息                                        |
| save-result        | false  | 是否打开执行结果存日志功能，打开之后所有命令的运行结果都将保存到`/home/admin/logs/arthas/arthas.log`中 |
| job-timeout        | 1d     | 异步后台任务的默认超时时间，超过这个时间，任务自动停止；比如设置 1d, 2h, 3m, 25s，分别代表天、小时、分、秒 |



## 系统信息

### dashboard

#### dashboard

```
[arthas@31358]$ dashboard
```



`dashboard` 命令可以查看当前系统的实时数据面板，这个面板会实时刷新。分为三个部分：第一部分为cpu监控，第二部分为内存监控，第三部分为环境信息



![1](../../../Image/2022/03/220317-1.png)



##### CPU监控

```
ID     NAME                                GROUP  PRIORITY STATE         %CPU DELTA_TIME TIME        INTERRUPTED DAEMON              
4065   System Clock                        main   5        TIMED_WAITING 1.28 0.064      5349:54.404 false       true                
-1     C2 CompilerThread0                  -      -1       -             0.11 0.005      2:22.809    false       true                
-1     VM Periodic Task Thread             -      -1       -             0.04 0.002      204:11.488  false       true                
-1     C1 CompilerThread1                  -      -1       -             0.02 0.001      1:37.857    false       true                
13     lettuce-timer-3-1                   main   5        TIMED_WAITING 0.02 0.000      41:39.644   false       false               
-1     VM Thread                           -      -1       -             0.01 0.000      14:12.641   false       true                
387993 arthas-NettyHttpTelnetBootstrap-3-2 system 5        RUNNABLE      0.01 0.000      0:0.327     false       true                
11     Catalina-utility-2                  main   1        TIMED_WAITING 0.01 0.000      14:18.970   false       false      
```



##### 内存监控

```
Memory                  used  total  max    usage    GC                                               
heap                    137M  158M   444M   30.98%   gc.copy.count                  32664                                                 
eden_space              33M   43M    122M   27.24%   gc.copy.time(ms)               216524                                           
survivor_space          1M    5M     15M    10.88%   gc.marksweepcompact.count      6                                                    
tenured_gen             102M  109M   306M   33.48%   gc.marksweepcompact.time(ms)   607
nonheap                 100M  143M   -1     69.93%
code_cache              8M    46M    240M   3.59%
metaspace               82M   86M    -1     94.78%
compressed_class_space  9M    10M    1024M  0.94%
direct                  80K   80K    -      100.00%
mapped                  0K    0K     -      0.00%
```



##### 环境信息

```                                                                                                                                    
os.name                   Linux
os.version                3.10.0-693.2.2.el7.x86_64
java.version              1.8.0_121
java.home                 /opt/jdk8/jdk1.8.0_121/jre
systemload.average        0.04
processors                1
timestamp/uptime          Thu Mar 17 18:30:46 CST 2022/35407181s 
```



输入 `q` 或者 `Ctrl+C` 可以退出dashboard命令。



### thread

#### thread

```
[arthas@31358]$ thread
```

​     

查看所有的线程信息和线程栈，包括总线程数、新线程数等信息和dashboard命令的第一部分。

![image-20220317195102741](../../../Image/2022/03/220317-2.png)



```
Threads Total: 53, NEW: 0, RUNNABLE: 13, BLOCKED: 0, WAITING: 25, TIMED_WAITING: 10, TERMINATED: 0, Internal threads: 5    
ID     NAME                                GROUP  PRIORITY STATE         %CPU DELTA_TIME TIME        INTERRUPTED DAEMON              
4065   System Clock                        main   5        TIMED_WAITING 1.28 0.064      5349:54.404 false       true                
-1     C2 CompilerThread0                  -      -1       -             0.11 0.005      2:22.809    false       true                
-1     VM Periodic Task Thread             -      -1       -             0.04 0.002      204:11.488  false       true                
-1     C1 CompilerThread1                  -      -1       -             0.02 0.001      1:37.857    false       true                
13     lettuce-timer-3-1                   main   5        TIMED_WAITING 0.02 0.000      41:39.644   false       false               
-1     VM Thread                           -      -1       -             0.01 0.000      14:12.641   false       true                
387993 arthas-NettyHttpTelnetBootstrap-3-2 system 5        RUNNABLE      0.01 0.000      0:0.327     false       true                
11     Catalina-utility-2                  main   1        TIMED_WAITING 0.01 0.000      14:18.970   false       false      
```



#### thread -n {num}

查看占用CPU最多的3个线程。

```
[arthas@31358]$ thread -n 3
```

 

#### thread pid

查询指定线程ID的线程信息。

```
[arthas@31358]$ thread 4065
```



查看线程ID为4065，名称为System Clock的详细信息。



#### thread -b

找出当前阻塞其他线程的线程。

```
[arthas@31358]$ thread -b
No most blocking thread found!
```



### jvm

#### jvm

直接输出当前jvm的各种信息。



```
[arthas@31358]$ jvm
```

```
 RUNTIME                                                                      
------------------------------------------------------------------------------
 MACHINE-NAME                31358@izuf6e56zt7ebjhby2hxboz                    
 JVM-START-TIME              2021-01-31 23:11:05                              
 MANAGEMENT-SPEC-VERSION     1.2                                              
 SPEC-NAME                   Java Virtual Machine Specification               
 SPEC-VENDOR                 Oracle Corporation                               
 SPEC-VERSION                1.8                                              
 VM-NAME                     Java HotSpot(TM) 64-Bit Server VM                
 VM-VENDOR                   Oracle Corporation                               
 VM-VERSION                  25.121-b13                                       
 INPUT-ARGUMENTS             []                                               
 CLASS-PATH                  tool-1.0-SNAPSHOT.jar                            
 BOOT-CLASS-PATH             /opt/jdk8/jdk1.8.0_121/jre/lib/resources.jar...
 LIBRARY-PATH                /usr/java/packages/lib/amd64:/usr/lib64:/lib64:/l
------------------------------------------------------------------------------
 CLASS-LOADING                                                                
------------------------------------------------------------------------------
 LOADED-CLASS-COUNT          15004                                            
 TOTAL-LOADED-CLASS-COUNT    15417                                            
 UNLOADED-CLASS-COUNT        413                                              
 IS-VERBOSE                  false                                            
------------------------------------------------------------------------------
 COMPILATION                                                                  
------------------------------------------------------------------------------
 NAME                        HotSpot 64-Bit Tiered Compilers                  
 TOTAL-COMPILE-TIME          138937                                           
 [time (ms)]                                                                  
------------------------------------------------------------------------------
 GARBAGE-COLLECTORS                                                           
------------------------------------------------------------------------------
 Copy                        name : Copy                                      
 [count/time (ms)]           collectionCount : 32915                          
                             collectionTime : 218175                          
 MarkSweepCompact            name : MarkSweepCompact                          
 [count/time (ms)]           collectionCount : 7                              
                             collectionTime : 808                             
------------------------------------------------------------------------------
 MEMORY-MANAGERS                                                              
------------------------------------------------------------------------------
 CodeCacheManager            Code Cache                                       
 Metaspace Manager           Metaspace                                        
                             Compressed Class Space                           
 Copy                        Eden Space                                       
                             Survivor Space                                   
 MarkSweepCompact            Eden Space                                       
                             Survivor Space                                   
                             Tenured Gen                                      
------------------------------------------------------------------------------
 MEMORY                                                                       
------------------------------------------------------------------------------
 HEAP-MEMORY-USAGE           init : 31457280(30.0 MiB)                        
 [memory in bytes]           used : 110951912(105.8 MiB)                      
                             committed : 166006784(158.3 MiB)                 
                             max : 466288640(444.7 MiB)                       
 NO-HEAP-MEMORY-USAGE        init : 2555904(2.4 MiB)                          
 [memory in bytes]           used : 122692048(117.0 MiB)                      
                             committed : 156434432(149.2 MiB)                 
                             max : -1(-1 B)                                   
 PENDING-FINALIZE-COUNT      0                                                
------------------------------------------------------------------------------
 OPERATING-SYSTEM                                                             
------------------------------------------------------------------------------
 OS                          Linux                                            
 ARCH                        amd64                                            
 PROCESSORS-COUNT            1                                                
 LOAD-AVERAGE                0.02                                             
 VERSION                     3.10.0-693.2.2.el7.x86_64                        
------------------------------------------------------------------------------
 THREAD                                                                       
------------------------------------------------------------------------------
 COUNT                       50                                               
 DAEMON-COUNT                45                                               
 PEAK-COUNT                  52                                               
 STARTED-COUNT               390936                                           
 DEADLOCK-COUNT              0                                                
------------------------------------------------------------------------------
 FILE-DESCRIPTOR                                                              
------------------------------------------------------------------------------
 MAX-FILE-DESCRIPTOR-COUNT   65535                                            
 OPEN-FILE-DESCRIPTOR-COUNT  82                                               
```



### sysprop

#### sysprop

查看所有的系统变量，也可以设置某个系统变量。



```
[arthas@31358]$ sysprop
```

```
 KEY                                   VALUE                                                                                                                                                
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 awt.toolkit                           sun.awt.X11.XToolkit                                                                                                                                 
 file.encoding.pkg                     sun.io                                                                                                                                               
 java.specification.version            1.8                                                                                                                                                  
 sun.cpu.isalist                                                                                                                                                                            
 sun.jnu.encoding                      UTF-8                                                                                                                                                
 java.class.path                       tool-1.0-SNAPSHOT.jar                                                                                                                                
 java.vm.vendor                        Oracle Corporation                                                                                                                                   
 sun.arch.data.model                   64                                                                                                                                                   
 java.vendor.url                       http://java.oracle.com/                                                                                                                              
 catalina.useNaming                    false                                                                                                                                                
 user.timezone                         Asia/Shanghai                                                                                                                                        
 os.name                               Linux                                                                                                                                                
 java.vm.specification.version         1.8                                                                                                                                                  
 user.country                          US                                                                                                                                                   
 sun.java.launcher                     SUN_STANDARD                                                                                                                                         
 sun.boot.library.path                 /opt/jdk8/jdk1.8.0_121/jre/lib/amd64                                                                                                                 
 sun.java.command                      tool-1.0-SNAPSHOT.jar                                                                                                                                
 sun.cpu.endian                        little                                                                                                                                               
 user.home                             /root                                                                                                                                                
 user.language                         en                                                                                                                                                   
 java.specification.vendor             Oracle Corporation                                                                                                                                   
 java.home                             /opt/jdk8/jdk1.8.0_121/jre                                                                                                                           
 file.separator                        /                                                                                                                                                    
 line.separator                                                                                                                                                                             
 java.vm.specification.vendor          Oracle Corporation                                                                                                                                   
 java.specification.name               Java Platform API Specification                                                                                                                      
 java.awt.graphicsenv                  sun.awt.X11GraphicsEnvironment                                                                                                                       
 java.awt.headless                     true                                                                                                                                                 
 sun.boot.class.path                   /opt/jdk8/jdk1.8.0_121/jre/lib/resources.jar:/opt/jdk8/jdk1.8.0_121/jre/lib/rt.jar:/opt/jdk8/jdk1.8.0_121/jre/lib/sunrsasign.jar:/opt/jdk8/jdk1.8.0_ 
                                       121/jre/lib/jsse.jar:/opt/jdk8/jdk1.8.0_121/jre/lib/jce.jar:/opt/jdk8/jdk1.8.0_121/jre/lib/charsets.jar:/opt/jdk8/jdk1.8.0_121/jre/lib/jfr.jar:/opt/ 
                                       jdk8/jdk1.8.0_121/jre/classes                                                                                                                        
 java.protocol.handler.pkgs            org.springframework.boot.loader                                                                                                                      
 sun.management.compiler               HotSpot 64-Bit Tiered Compilers                                                                                                                      
 java.runtime.version                  1.8.0_121-b13                                                                                                                                        
 user.name                             root                                                                                                                                                 
 path.separator                        :                                                                                                                                                    
 os.version                            3.10.0-693.2.2.el7.x86_64                                                                                                                            
 java.endorsed.dirs                    /opt/jdk8/jdk1.8.0_121/jre/lib/endorsed                                                                                                              
 java.runtime.name                     Java(TM) SE Runtime Environment                                                                                                                      
 file.encoding                         UTF-8                                                                                                                                                
 spring.beaninfo.ignore                true                                                                                                                                                 
 sun.nio.ch.bugLevel                                                                                                                                                                        
 java.vm.name                          Java HotSpot(TM) 64-Bit Server VM                                                                                                                    
 java.vendor.url.bug                   http://bugreport.sun.com/bugreport/                                                                                                                  
 java.io.tmpdir                        /tmp                                                                                                                                                 
 catalina.home                         /tmp/tomcat.5486222080369585181.8002                                                                                                                 
 com.zaxxer.hikari.pool_number         1                                                                                                                                                    
 java.version                          1.8.0_121                                                                                                                                            
 user.dir                              /opt                                                                                                                                                 
 os.arch                               amd64                                                                                                                                                
 PID                                   31358                                                                                                                                                
 java.vm.specification.name            Java Virtual Machine Specification                                                                                                                   
 java.awt.printerjob                   sun.print.PSPrinterJob                                                                                                                               
 sun.os.patch.level                    unknown                                                                                                                                              
 catalina.base                         /tmp/tomcat.5486222080369585181.8002                                                                                                                 
 java.library.path                     /usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib                                                                                         
 java.vm.info                          mixed mode                                                                                                                                           
 java.vendor                           Oracle Corporation                                                                                                                                   
 java.vm.version                       25.121-b13                                                                                                                                           
 java.ext.dirs                         /opt/jdk8/jdk1.8.0_121/jre/lib/ext:/usr/java/packages/lib/ext                                                                                        
 sun.io.unicode.encoding               UnicodeLittle                                                                                                                                        
 java.class.version                    52.0    
```



### sysenv

#### sysenv

查看所有的操作系统环境变量，也可以查看设置某个环境变量。



```
[arthas@31358]$ sysenv
```

```
 KEY                                   VALUE                                                                                                                                                
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 PATH                                  /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/jdk8/jdk1.8.0_121/bin:/root/bin                                                               
 HISTCONTROL                           ignoredups                                                                                                                                           
 HISTSIZE                              1000                                                                                                                                                 
 JAVA_HOME                             /opt/jdk8/jdk1.8.0_121                                                                                                                               
 TERM                                  xterm                                                                                                                                                
 XFILESEARCHPATH                       /usr/dt/app-defaults/%L/Dt                                                                                                                           
 LANG                                  en_US.UTF-8                                                                                                                                          
 XDG_SESSION_ID                        34097                                                                                                                                                
 MAIL                                  /var/spool/mail/root                                                                                                                                 
 LOGNAME                               root                                                                                                                                                 
 PWD                                   /opt                                                                                                                                                 
 _                                     /usr/bin/nohup                                                                                                                                       
 LESSOPEN                              ||/usr/bin/lesspipe.sh %s                                                                                                                            
 SHELL                                 /bin/bash                                                                                                                                            
 SSH_TTY                               /dev/pts/5                                                                                                                                           
 SSH_CLIENT                            113.88.237.34 32215 22                                                                                                                               
 OLDPWD                                /root                                                                                                                                                
 USER                                  root                                                                                                                                                 
 CLASSPATH                             :/opt/jdk8/jdk1.8.0_121/lib/                                                                                                                         
 SSH_CONNECTION                        113.88.237.34 32215 172.19.104.23 22                                                                                                                 
 HOSTNAME                              izuf6e56zt7ebjhby2hxboz                                                                                                                              
 NLSPATH                               /usr/dt/lib/nls/msg/%L/%N.cat                                                                                                                        
 XDG_RUNTIME_DIR                       /run/user/0                                                                                                                                          
 LS_COLORS                             rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;05;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34 
                                       ;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz 
                                       =01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.bz2=01;31:*.bz= 
                                       01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;3 
                                       1:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*. 
                                       tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mp 
                                       eg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf= 
                                       01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35: 
                                       *.emf=01;35:*.axv=01;35:*.anx=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.m 
                                       pc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.axa=01;36:*.oga=01;36:*.spx=01;36:*.xspf=01;36:                                                        
 HOME                                  /root                                                                                                                                                
 SHLVL                                 1                                                                           
```





## 类信息

### sc         

通过sc可以查看已加载类的相关信息，比如该类是从哪个jar包加载的，被哪个类加载器加载的，以及是否是接口等等。



#### sc  {class-pattern}

查找JVM里已加载的类

```
[arthas@31358]$ sc com.tool.bookmark.controller.BookmarkController
```

```
com.tool.bookmark.controller.BookmarkController
com.tool.bookmark.controller.BookmarkController$$EnhancerBySpringCGLIB$$8b0896ae
Affect(row-cnt:2) cost in 22 ms.
```





#### sc -d {class-pattern}

查看JVM里已加载的指定类的详细信息，比如该类是从哪个jar包加载的，被哪个类加载器加载的，以及是否是接口等等。

```
[arthas@31358]$ sc -d com.tool.bookmark.controller.BookmarkController
```

```
class-info        com.tool.bookmark.controller.BookmarkController    
code-source       file:/opt/tool-1.0-SNAPSHOT.jar!/BOOT-INF/classes!/
name              com.tool.bookmark.controller.BookmarkController    
isInterface       false                                              
isAnnotation      false                                              
isEnum            false                                              
isAnonymousClass  false                                              
isArray           false                                              
isLocalClass      false                                              
isMemberClass     false                                              
isPrimitive       false                                              
isSynthetic       false                                              
simple-name       BookmarkController                                 
modifier          public                                             
annotation        org.springframework.web.bind.annotation.CrossOrigin,
               org.springframework.web.bind.annotation.RestController,
               org.springframework.web.bind.annotation.RequestMapping
interfaces                                                                         
super-class       +-java.lang.Object                                               
class-loader      +-org.springframework.boot.loader.LaunchedURLClassLoader@5197848c
                 +-sun.misc.Launcher$AppClassLoader@42a57993                    
                   +-sun.misc.Launcher$ExtClassLoader@206a70ef                  
classLoaderHash   5197848c  
```



### sm         

#### sm {class-pattern}

sm命令表示查看已加载类的方法详情。



```
[arthas@31358]$ sm com.tool.bookmark.entity.pojo.BookmarkClassify
```

```
com.tool.bookmark.entity.pojo.BookmarkClassify <init>()V
com.tool.bookmark.entity.pojo.BookmarkClassify getCount()Ljava/lang/Integer;
com.tool.bookmark.entity.pojo.BookmarkClassify getSort()Ljava/lang/Integer;
com.tool.bookmark.entity.pojo.BookmarkClassify setId(Ljava/lang/String;)Lcom/tool/bookmark/entity/pojo/BookmarkClassify;
com.tool.bookmark.entity.pojo.BookmarkClassify setCount(Ljava/lang/Integer;)Lcom/tool/bookmark/entity/pojo/BookmarkClassify;
com.tool.bookmark.entity.pojo.BookmarkClassify setTitle(Ljava/lang/String;)Lcom/tool/bookmark/entity/pojo/BookmarkClassify;
com.tool.bookmark.entity.pojo.BookmarkClassify setSort(Ljava/lang/Integer;)Lcom/tool/bookmark/entity/pojo/BookmarkClassify;
com.tool.bookmark.entity.pojo.BookmarkClassify canEqual(Ljava/lang/Object;)Z
com.tool.bookmark.entity.pojo.BookmarkClassify getCreateTime()Ljava/util/Date;
com.tool.bookmark.entity.pojo.BookmarkClassify getUpdateTime()Ljava/util/Date;
com.tool.bookmark.entity.pojo.BookmarkClassify getIsDeleted()Ljava/lang/Boolean;
com.tool.bookmark.entity.pojo.BookmarkClassify setCreateTime(Ljava/util/Date;)Lcom/tool/bookmark/entity/pojo/BookmarkClassify;
com.tool.bookmark.entity.pojo.BookmarkClassify setUpdateTime(Ljava/util/Date;)Lcom/tool/bookmark/entity/pojo/BookmarkClassify;
com.tool.bookmark.entity.pojo.BookmarkClassify setIsDeleted(Ljava/lang/Boolean;)Lcom/tool/bookmark/entity/pojo/BookmarkClassify;
com.tool.bookmark.entity.pojo.BookmarkClassify getTitle()Ljava/lang/String;
com.tool.bookmark.entity.pojo.BookmarkClassify getUserId()Ljava/lang/String;
com.tool.bookmark.entity.pojo.BookmarkClassify setUserId(Ljava/lang/String;)Lcom/tool/bookmark/entity/pojo/BookmarkClassify;
com.tool.bookmark.entity.pojo.BookmarkClassify equals(Ljava/lang/Object;)Z
com.tool.bookmark.entity.pojo.BookmarkClassify toString()Ljava/lang/String;
com.tool.bookmark.entity.pojo.BookmarkClassify hashCode()I
com.tool.bookmark.entity.pojo.BookmarkClassify getId()Ljava/lang/String;
Affect(row-cnt:21) cost in 13 ms.

```



### dump

#### dump

将已加载类的字节码dump到本地磁盘上。



### jad        

#### jad {class-pattern}

反编译代码，查看当前JVM中类的代码。此命令在项目发布定位某个类信息有没有更改过来时，使用这个命令可以很快定位代码是否修改。

```
[arthas@31358]$ jad com.tool.bookmark.controller.BookmarkController
```

```
ClassLoader:                                                     
+-org.springframework.boot.loader.LaunchedURLClassLoader@5197848c
  +-sun.misc.Launcher$AppClassLoader@42a57993                    
    +-sun.misc.Launcher$ExtClassLoader@206a70ef                  
Location:                                                        
file:/opt/tool-1.0-SNAPSHOT.jar!/BOOT-INF/classes!/              
       /*
        * Decompiled with CFR.
        * 
        * Could not load the following classes:
        *  com.baomidou.mybatisplus.extension.plugins.pagination.Page
				*  ...
        *  org.springframework.web.multipart.MultipartFile
        */
       package com.tool.bookmark.controller;
       
       import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
       ...
       import org.springframework.web.multipart.MultipartFile;
       
       @CrossOrigin
       @RestController
       @RequestMapping(value={"/bookmark"})
       public class BookmarkController {
           private static final Logger log = LoggerFactory.getLogger(BookmarkController.class);
           private IBookmarkService bookMarkService;
       
           @GetMapping(value={"/get-title-of-url"})
           public R<String> getTitleOfUrl(@RequestParam(value="url") String url) {
/*39*/         return this.bookMarkService.getTitleOfUrl(url);
           }
       }

Affect(row-cnt:4) cost in 1230 ms.
```



### classloader

#### classloader

将 JVM 中所有的classloader的信息统计出来，并可以展示继承树，urls等。

```
[arthas@31358]$ classloader
```

```
name                                                    numberOfInstances  loadedCountTotal
org.springframework.boot.loader.LaunchedURLClassLoader  1                  9385            
BootstrapClassLoader                                    1                  3741            
com.taobao.arthas.agent.ArthasClassloader               1                  2623            
sun.reflect.DelegatingClassLoader                       205                205             
sun.misc.Launcher$ExtClassLoader                        1                  74              
sun.misc.Launcher$AppClassLoader                        1                  49              
com.alibaba.fastjson.util.ASMClassLoader                1                  1         
...      
Affect(row-cnt:8) cost in 15 ms.
```





### redefine

#### redefine

该命令可以加载外部的.class文件，然后覆盖 jvm已加载的类。注意，这个命令不一定都能覆盖成功，如果添加了新的field，就不会加载成功。



```

```



#### 参考资料

- [Arthas实践--使用redefine排查应用奇怪的日志来源 ](https://github.com/alibaba/arthas/issues/263)



### getstatic  

#### getstatic

查看类的静态属性。

```
[arthas@31358]$ getstatic com.tool.user.consts.UserConstant AVATAR
```

```
field: AVATAR
@String[https://online-teaching.oss-cn-shenzhen.aliyuncs.com/dog.jpg]
Affect(row-cnt:1) cost in 10 ms.
```



**查看类的静态属性ognl命令也可以做到，官方也比较推荐使用ognl表达式来做**。



## 方法监听

### monitor    

#### monitor

monitor命令可以监控方法的执行情况。比如调用成功次数，失败次数，失败率、平均执行时间等等。默认120秒输出一次，也就是说，当我们输入monitor命令之后，每120秒就会输出一次统计结果。

通过-c参数可以修改输出频率，支持通配符和正则表达式。



```
[arthas@31358]$ monitor com.tool.bookmark.controller.BookmarkController list
```

```
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 159 ms, listenerId: 11
 timestamp            class                                                  method total success fail avg-rt(ms) fail-rate
---------------------------------------------------------------------------------------------------------------------------
 2022-03-21 21:25:51  com.tool.bookmark.controller.BookmarkController        list   2     2       0    41.05      0.00%    
 2022-03-21 21:25:51  com.tool.bookmark.controller.BookmarkController$$Enha  list   2     2       0    43.67      0.00%    
                      ncerBySpringCGLIB$$8b0896ae                                                                          

```



#### 后台异步任务

当线上出现偶发的问题，比如需要watch某个条件，而这个条件一天可能才会出现一次时，异步后台任务就派上用场了。

运行异步任务并将结果重定向到指定文件：



```
monitor *.Test test -c >> /tmp/monitor.log &
# 后面通过jobs查看
jobs
# 通过kill 终止任务
kill 1
# 通过fg将任务拉到前台运行
fg 1
# 按下 clrt + z 后又返回后台运行
```



- 使用 > 将结果重写向到日志文件，使用 & 指定命令是后台运行，session断开不影响任务执行（生命周期默认为1天）
- jobs——列出所有job
- kill——强制终止任务
- fg——将暂停的任务拉到前台执行
- bg——将暂停的任务放到后台执行



> 最多同时支持8个命令使用重定向将结果写日志
>
> 请勿同时开启过多的后台异步命令，以免对目标JVM性能造成影响



### watch    

#### watch {class-pattern} {method-pattern}

通过`watch`命令可以查看函数的参数/返回值/异常信息。

```
[arthas@31358]$ watch com.tool.bookmark.controller.BookmarkController list
```

```
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 215 ms, listenerId: 8
method=com.tool.bookmark.controller.BookmarkController.list location=AtExit
ts=2022-03-17 22:34:15; [cost=44.629703ms] result=@ArrayList[
    @Object[][isEmpty=false;size=2],
    @BookmarkController[com.tool.bookmark.controller.BookmarkController@12c30b9e],
    @R[R(code=200, success=true, data=com.baomidou.mybatisplus.extension.plugins.pagination.Page@64b9fbdb, msg=操作成功)],
]
method=com.tool.bookmark.controller.BookmarkController$$EnhancerBySpringCGLIB$$8b0896ae.list location=AtExit
ts=2022-03-17 22:34:15; [cost=48.357729ms] result=@ArrayList[
    @Object[][isEmpty=false;size=2],
    @BookmarkController$$EnhancerBySpringCGLIB$$8b0896ae[com.tool.bookmark.controller.BookmarkController@12c30b9e],
    @R[R(code=200, success=true, data=com.baomidou.mybatisplus.extension.plugins.pagination.Page@64b9fbdb, msg=操作成功)],
]
```



- 让你能方便的观察到指定方法的调用情况。能观察到的范围为：返回值、抛出异常、入参，通过编写 OGNL 表达式进行对应变量的查看。
- watch的使用姿势比较丰富，可以在四个不同的场景观察方法的执行。比如方法调用之前、方法调用之后、方法异常之后、方法结束之后。默认观察的是方法结束之后。
- 如果观察的是方法结束之后的场景，由于入参可能在执行方法时被改变，所以此时输出的可能不是真正的入参。因此，要看真正的入参，要看方法调用之前的，也就是加上-b的参数。
- 另外，使用-b参数观察的话，则观察不到方法返回的结果以及抛出的异常了。

| 参数名称          | 参数说明                                      |
| ----------------- | --------------------------------------------- |
| class-pattern     | 类名表达式匹配                                |
| method-pattern    | 方法名表达式匹配                              |
| express           | 观察表达式                                    |
| condition-express | 条件表达式                                    |
| [b]               | 在方法调用之前观察                            |
| [e]               | 在方法异常之后观察                            |
| [s]               | 在方法返回之后观察                            |
| [f]               | 在方法结束之后(正常返回和异常返回)观察        |
| [E]               | 开启正则表达式匹配，默认为通配符匹配          |
| [x:]              | 指定输出结果的属性遍历深度，默认为 1          |
| [n:]              | 只执行n次，默认会不断输出，除非用户按下cltr+c |

```bash
# 观察CommonTest的test方法
# 输出 入参、返回结果、抛出的异常 —— 输出的内容可以动态调整
# 后面跟着的是 条件表达式，表示耗时超过10ms才输出
# -n 表示只执行一次，-x表示 入参和返回结果的展开层次为5层
watch *.CommonTest test "{params,returnObj,throwExp}" '#cost>10' -x 5 -n 1

# 耗时大于10ms并且第一个参数等于1才输出
watch *.CommonTest test "{params,returnObj,throwExp}" '#cost>10 && params[0]==1' -x 5 -n 1
# 第一个参数大于1 并且第二个参数等于hello才输出
watch *.CommonTest test "{params,returnObj,throwExp}" 'params[0]>1 && params[1]=="hello"' -x 5 -n 1
# 第一个参数小于5或者第二个参数等于"world"就输出
watch *.CommonTest test "{params,returnObj,throwExp}" 'params[0]<5 || params[1]=="wolrd"' -x 5 -n 1
# 第一个参数的name字段等于world时才输出。
# 由于在方法执行过程中参数的name属性可能发生改变，因此加上-b才能观察到真正的入参
watch -b *.CommonTest test "{params,returnObj,throwExp}" 'params[0].name=="wolrd"' -x 5 -n 1

# 由于同时指定了-s和-b，所以方法被调用一次，就会输出2次结果(两个场景分开输出)，分别是方法被调用前，和返回之后
# 注意，这里如果-n只设置成1，那么只会输出-b对应的输出,-s对应的输出由于没有次数了就无法输出了
watch *.CommonTest test '{params,returnObj,throwExp}' -x 5 -n 2 -s -b
```

在填写条件表达式时要注意一点，条件表达式中的params默认都是获取的方法执行完后的参数信息，比如入参a的属性name方法执行前是"hello",在方法执行后变成了"world",那么条件表达式传入'params[0].name="hello"'将不会输出，只有填入'params[0].name="hello"'才可以匹配上。这点对于后面的trace、stack命令也是一样的。

查看方法输入参数/返回值/异常信息，watch demo.MathGame primeFactors "{params,returnObj}" -x 2



### trace  

#### trace {class-pattern} {method-pattern}

方法内部调用路径，并输出方法路径上的每个节点上耗时，tt命令会记录每次方法调用的各种信息。它和watch有些相似但是它能记录下各个时间点的调用信息，之后随时查看，甚至replay这次调用。

但是该命令只能输出一级调用的方法耗时，往下的就不会输出了。



```
[arthas@31358]$ trace com.tool.bookmark.controller.BookmarkController list
```

```
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 197 ms, listenerId: 9
`---ts=2022-03-20 20:49:05;thread_name=http-nio-8002-exec-4;id=15;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@2d1ef81a
    `---[60.537798ms] com.tool.bookmark.controller.BookmarkController$$EnhancerBySpringCGLIB$$8b0896ae:list()
        `---[60.462193ms] org.springframework.cglib.proxy.MethodInterceptor:intercept()
            `---[50.653487ms] com.tool.bookmark.controller.BookmarkController:list()
                +---[50.572592ms] com.tool.bookmark.service.IBookmarkService:list() #33
                `---[0.017509ms] com.tool.api.result.R:data() #34
```



> trace 在执行的过程中本身是会有一定的性能开销，在统计的报告中并未像 JProfiler 一样预先减去其自身的统计开销。所以这统计出来有些许的不准，渲染路径上调用的类、方法越多，性能偏差越大。但还是能让你看清一些事情的。
>
> 这里存在一个统计不准确的问题，就是所有方法耗时加起来可能会小于该监测方法的总耗时，这个是由于 Arthas 本身的逻辑会有一定的耗时。



### stack    

输出当前方法被调用的调用路径。用法和trace差不多。

```
[arthas@31358]$ stack com.tool.bookmark.controller.BookmarkController list
```

```
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 192 ms, listenerId: 12
ts=2022-03-21 21:32:31;thread_name=http-nio-8002-exec-10;id=1b;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@2d1ef81a
    @com.tool.bookmark.controller.BookmarkController.list()
        at com.tool.bookmark.controller.BookmarkController$$FastClassBySpringCGLIB$$fa77da95.invoke(<generated>:-1)
        at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:769)
        at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163)
        at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:747)
        at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:88)
        at com.tool.log.aspect.RequestAspectLog.aroundApi(RequestAspectLog.java:90)
        at sun.reflect.GeneratedMethodAccessor82.invoke(null:-1)
				...
        at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:202)
        at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)
				...
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
        at java.lang.Thread.run(Thread.java:745)
```



### tt 

#### tt {class-pattern} {method-pattern}

timetunnel，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测，同时可回放该方法调用。



```
[arthas@31358]$ tt -t com.tool.bookmark.controller.BookmarkController list
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 167 ms, listenerId: 10
 INDEX  TIMESTAMP            COST(ms)   IS-RET  IS-EXP  OBJECT      CLASS                                                METHOD
---------------------------------------------------------------
 1000   2022-03-20 20:51:07  37.838183  true    false   0x12c30b9e  BookmarkController                                   list  
 1001   2022-03-20 20:51:07  47.93884   true    false   0x69f14919  BookmarkController$$EnhancerBySpringCGLIB$$8b0896ae  list  
```



#### tt {class-pattern} {method-pattern} pattern

记录指定条件下的调用信息。

```
[arthas@31358]$ tt -t com.tool.bookmark.controller.BookmarkController list params[0].current==2
```

```
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 206 ms, listenerId: 15
 INDEX  TIMESTAMP            COST(ms)   IS-RET  IS-EXP  OBJECT      CLASS                                                METHOD
---------------------------------------------------------------
 1000   2022-03-20 20:51:07  37.838183  true    false   0x12c30b9e  BookmarkController                                   list  
 1001   2022-03-20 20:51:07  47.93884   true    false   0x69f14919  BookmarkController$$EnhancerBySpringCGLIB$$8b0896ae  list  
```



#### tt -i {INDEX} -p

重试INDEX为1000的调用。

```
[arthas@31358]$ tt -i 1002 -p	
```

```
RE-INDEX       1002                                                                                 
GMT-REPLAY     2022-03-21 21:38:20                                                                  
OBJECT         0x12c30b9e                                                                           
CLASS          com.tool.bookmark.controller.BookmarkController                                      
METHOD         list                                                                                 
PARAMETERS[0]  @Page[                                                                               
                   serialVersionUID=@Long[8545996863226528798],                                     
                   records=@ArrayList[isEmpty=false;size=8],                                        
                   total=@Long[61],                                                                 
                   size=@Long[8],                                                                   
                   current=@Long[1],                                                                
                   orders=@ArrayList[isEmpty=true;size=0],                                          
                   optimizeCountSql=@Boolean[true],                                                 
                   isSearchCount=@Boolean[true],                                                    
               ]                                                                                    
PARAMETERS[1]  @Bookmark[                                                                           
                   serialVersionUID=@Long[1],                                                       
                   id=null,                                                                         
                   title=@String[],                                                                 
                   url=null,                                                                        
                   status=null,                                                                     
                   userId=null,                                                                     
                   classifyId=@String[1174928055923408897],                                         
                   sort=null,                                                                       
                   isDeleted=null,                                                                  
                   createTime=null,                                                                 
                   updateTime=null,                                                                 
               ]                                                                                    
IS-RETURN      true                                                                                 
IS-EXCEPTION   false                                                                                
COST(ms)       34.468322                                                                            
RETURN-OBJ     @R[                                                                                  
                   serialVersionUID=@Long[1],                                                       
                   code=@Integer[200],                                                              
                   success=@Boolean[true],                                                          
                   data=@Page[com.baomidou.mybatisplus.extension.plugins.pagination.Page@2dadeec8], 
                   msg=@String[操作成功],                                                               
               ]                                                                                    
Time fragment[1002] successfully replayed 1 times.
```



> 使用tt进行方法replay时，要注意2点
>
> ThreadLocal 信息丢失
>
> 很多框架偷偷的将一些环境变量信息塞到了发起调用线程的 ThreadLocal 中，由于调用线程发生了变化，这些 ThreadLocal 线程信息无法通过 Arthas 保存，所以这些信息将会丢失。
>
> 一些常见的 CASE 比如：鹰眼的 TraceId 等。
>
> 引用的对象
>
> 需要强调的是，tt 命令是将当前环境的对象引用保存起来，但仅仅也只能保存一个引用而已。如果方法内部对入参进行了变更，或者返回的对象经过了后续的处理，那么在 tt 查看的时候将无法看到当时最准确的值。这也是为什么 watch 命令存在的意义。



#### tt -l

输出之前记录的调用。

```
[arthas@31358]$ tt -l
```

```
 INDEX  TIMESTAMP            COST(ms)   IS-RET  IS-EXP  OBJECT      CLASS                                                METHOD
-------------------------------------------------------------------------------------------------------------------------------
 1000   2022-03-20 20:51:07  37.838183  true    false   0x12c30b9e  BookmarkController                                   list  
 1001   2022-03-20 20:51:07  47.93884   true    false   0x69f14919  BookmarkController$$EnhancerBySpringCGLIB$$8b0896ae  list  
 1002   2022-03-21 21:38:02  51.113143  true    false   0x12c30b9e  BookmarkController                                   list  
 1003   2022-03-21 21:38:02  54.220929  true    false   0x69f14919  BookmarkController$$EnhancerBySpringCGLIB$$8b0896ae  list  
 1004   2022-03-21 21:42:34  29.347765  true    false   0x12c30b9e  BookmarkController                                   list  
 1005   2022-03-21 21:42:34  36.793334  true    false   0x69f14919  BookmarkController$$EnhancerBySpringCGLIB$$8b0896ae  list  
 1006   2022-03-21 21:42:45  29.066651  true    false   0x12c30b9e  BookmarkController                                   list  
 1007   2022-03-21 21:42:45  30.901619  true    false   0x69f14919  BookmarkController$$EnhancerBySpringCGLIB$$8b0896ae  list  
Affect(row-cnt:8) cost in 0 ms.
```



### ognl  

#### ognl

可执行任意代码



#### 参考资料

- [【Arthas问题排查集】活用ognl表达式](https://github.com/alibaba/arthas/issues/11)



## 其他

### auth      

### keymap       

### memory     
### perfcounter 
### mc           
### retransform
### dump       

### heapdump         
### vmoption   
### logger       
### cat        
### base64     
### echo       
### pwd        
### mbean      
### grep       

Arthas支持使用管道对上述命令的结果进行进一步的处理。如`sm java.lang.String * | grep 'index'`

- grep——搜索满足条件的结果
- plaintext——将命令的结果去除ANSI颜色
- wc——按行统计输出结果



### tee        
### profiler   
### vmtool     






# 参考资料

- [官方文档](https://arthas.aliyun.com/doc/install-detail.html)
- [下线教程](https://arthas.aliyun.com/doc/arthas-tutorials.html?language=cn) 



