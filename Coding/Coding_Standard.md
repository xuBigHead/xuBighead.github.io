# 目录

[TOC]
# 整洁代码规范
## 为什么要编写整洁的代码
对于具备一定开发经验的开发者来说，编写整洁代码的原因无需多言。

**有数据显示读代码的时间与写代码的时间比例超过 10：1，并且编写当前代码的难度，取决于读周边代码的难度。**



## 好代码的特征
好的代码应该具备：易拓展和维护、简洁（只做好一件事）、可复用性强（没有重复代码）、能快速写出单元测试。可读性强、没有副作用（做了名称以外的工作）。




## 编码规范
### 魔法数
不要使用无业务含义的数字、字符，例如数字86400应该用常量SECONDS_PER_DAY来表达；
```java
@Test
public void forbidMagicWord(){
    int day = 10;
    // 乘以86400时，没有给这个数字赋予一个含义，导致不知道为什么要乘以这个数
    int secondsOfDay = day * 86400;
    
    // 通过声明一个名称有意义的变量来赋值86400，来使86400表示的是一天中的秒数
    int secondsPerDay = 86400;
    int rightSecondsOfDay = day * secondsPerDay;
}
```



## 命名规范

### 使用生僻字，又臭又长

如类名为UltimateAssociatedSubjectRunBatchServiceImpl的类，读起来就不方便，尽量使用满足含义的简单名称。



### 名不副实

命名时的名称与之对应的功能不匹配，不能表述或错误的表述了对应的功能。



### 结合上下文简化名称

在命名时，要结合所处的上下文，来合理的简化变量或方法的命名。



```java
public class User {
    /**
     * 这个成员变量表示的手机号，因为其位于User类下，因此可以简化为mobile，就能表示是用户手机号。
     */
    private String userMobile;
    private String mobile;
}
```



### 常见命名规范

- 函数的命名要体现做什么，而不是怎么做；

- 对于辅助类，尽量不要用Helper、Util之类的后缀，因为其含义太过笼统，容易破坏SRP（单一职责原则）；

- 保持命名的一致性，可以提高代码的可读性，从而简化复杂度；


| 类型 | 命令标识 | 类型             | 命令标识 |
| ---- | -------- | ---------------- | -------- |
| 新增 | create   | 查询（单个结果） | get      |
| 添加 | add      | 查询（多个结果） | list     |
| 删除 | remove   | 分页查询         | page     |
| 修改 | update   | 统计             | count    |



- 遵守对仗词的命名规则有助于保持一致性，从而提高代码的可读性，如下所示；
| 对仗词A | 对仗词B | 对仗词A   | 对仗词B   | 对仗词A | 对仗词B |
| ------- | ------- | --------- | --------- | ------- | ------- |
| add     | remove  | increment | decrement | open    | close   |
| begin   | end     | insert    | delete    | show    | hide    |
| create  | destroy | lock      | unlock    | source  | target  |
| first   | last    | min       | max       | start   | stop    |
| get     | set     | next      | previous  | up      | down    |
| old     | new     |           |           |         |         |

- 把限定词加到名字的最后，并在项目中贯彻执行，保持命名风格的一致性；



| Controller     | Biz      | Service | Mapper |
| -------------- | -------- | ------- | ------ |
| get*query      | doGet    | get     | select |
| save*command   | doSave   | save    | insert |
| modify*command | doModify | modify  | update |
| remove*command | doRemove | remove  | delete |



#### 类名

**类名使用大驼峰命名形式**，应该使用**名词或者名词短语**



| 属性           | 约束                                  | 例                                                           |
| :------------- | :------------------------------------ | :----------------------------------------------------------- |
| 抽象类         | Abstract 或者 Base 开头               | BaseUserService                                              |
| 枚举类         | Enum 作为后缀                         | GenderEnum                                                   |
| 工具类         | Utils作为后缀                         | StringUtils                                                  |
| 异常类         | Exception结尾                         | RuntimeException                                             |
| 接口实现类     | 接口名+ ImpI 或者 前缀接口名 + 接口名 | UserService + UserServiceImpl、IUserService + UserService    |
| 领域模型相关   | /DO/DTO/VO/DAO                        | 正例：UserDAO 反例：UserDo， UserDao                         |
| 设计模式相关类 | Builder，Factory等                    | 当使用到设计模式时，需要使用对应的设计模式作为后缀，如ThreadFactory |
| 处理特定功能的 | Handler，Predicate, Validator         | 表示处理器，校验器，断言，这些类工厂还有配套的方法名如handle，predicate，validate |
| 测试类         | Test结尾                              | UserServiceTest， 表示用来测试UserService类的                |



#### 方法名

方法命名一般为**动词或动词短语**，与参数或参数名共同组成动宾短语，即动词 + 名词。



##### 布尔返回值的方法

| 位置   | 单词   | 意义                                                         | 例            |
| :----- | :----- | :----------------------------------------------------------- | :------------ |
| Prefix | is     | 对象是否符合期待的状态                                       | isValid       |
| Prefix | can    | 对象**能否执行**所期待的动作                                 | canRemove     |
| Prefix | should | 调用方执行某个命令或方法是**好还是不好**,**应不应该**，或者说**推荐还是不推荐** | shouldMigrate |
| Prefix | has    | 对象**是否持有**所期待的数据和属性                           | hasObservers  |
| Prefix | needs  | 调用方**是否需要**执行某个命令或方法                         | needsMigrate  |



##### 按需执行的方法

| 位置   | 单词      | 意义                                      | 例                     |
| :----- | :-------- | :---------------------------------------- | :--------------------- |
| Suffix | IfNeeded  | 需要的时候执行，不需要的时候什么都不做    | drawIfNeeded           |
| Prefix | might     | 同上                                      | mightCreate            |
| Prefix | try       | 尝试执行，失败时抛出异常或是返回errorcode | tryCreate              |
| Suffix | OrDefault | 尝试执行，失败时返回默认值                | getOrDefault           |
| Suffix | OrElse    | 尝试执行、失败时返回实际参数中指定的值    | getOrElse              |
| Prefix | force     | 强制尝试执行。error抛出异常或是返回值     | forceCreate, forceStop |



##### 用来检查的方法

| 单词     | 意义                                                 | 例             |
| :------- | :--------------------------------------------------- | :------------- |
| ensure   | 检查是否为期待的状态，不是则抛出异常或返回error code | ensureCapacity |
| validate | 检查是否为正确的状态，不是则抛出异常或返回error code | validateInputs |



##### 异步相关方法

| 位置            | 单词         | 意义                                         | 例                    |
| :-------------- | :----------- | :------------------------------------------- | :-------------------- |
| Prefix          | blocking     | 线程阻塞方法                                 | blockingGetUser       |
| Suffix          | InBackground | 执行在后台的线程                             | doInBackground        |
| Suffix          | Async        | 异步方法                                     | sendAsync             |
| Suffix          | Sync         | 对应已有异步方法的同步方法                   | sendSync              |
| Prefix or Alone | schedule     | Job和Task放入队列                            | schedule, scheduleJob |
| Prefix or Alone | post         | 同上                                         | postJob               |
| Prefix or Alone | execute      | 执行异步方法（注：我一般拿这个做同步方法名） | execute, executeTask  |
| Prefix or Alone | start        | 同上                                         | start, startJob       |
| Prefix or Alone | cancel       | 停止异步方法                                 | cancel, cancelJob     |
| Prefix or Alone | stop         | 同上                                         | stop, stopJob         |



##### 回调方法

| 位置   | 单词   | 意义                       | 例           |
| :----- | :----- | :------------------------- | :----------- |
| Prefix | on     | 事件发生时执行             | onCompleted  |
| Prefix | before | 事件发生前执行             | beforeUpdate |
| Prefix | pre    | 同上                       | preUpdate    |
| Prefix | will   | 同上                       | willUpdate   |
| Prefix | after  | 事件发生后执行             | afterUpdate  |
| Prefix | post   | 同上                       | postUpdate   |
| Prefix | did    | 同上                       | didUpdate    |
| Prefix | should | 确认事件是否可以发生时执行 | shouldUpdate |



##### 操作对象生命周期的方法

| 单词       | 意义                           | 例              |
| :--------- | :----------------------------- | :-------------- |
| initialize | 初始化。也可作为延迟初始化使用 | initialize      |
| pause      | 暂停                           | onPause ，pause |
| stop       | 停止                           | onStop，stop    |
| abandon    | 销毁的替代                     | abandon         |
| destroy    | 同上                           | destroy         |
| dispose    | 同上                           | dispose         |



##### 与集合操作相关的方法

| contains | 是否持有与指定对象相同的对象 | contains   |
| -------- | ---------------------------- | ---------- |
| add      | 添加                         | addJob     |
| append   | 添加                         | appendJob  |
| insert   | 插入到下标n                  | insertJob  |
| put      | 添加与key对应的元素          | putJob     |
| remove   | 移除元素                     | removeJob  |
| enqueue  | 添加到队列的最末位           | enqueueJob |
| dequeue  | 从队列中头部取出并移除       | dequeueJob |
| push     | 添加到栈头                   | pushJob    |
| pop      | 从栈头取出并移除             | popJob     |
| peek     | 从栈头取出但不移除           | peekJob    |
| find     | 寻找符合条件的某物           | findById   |



##### 与数据相关的方法

| 单词   | 意义                                   | 例            |
| :----- | :------------------------------------- | :------------ |
| create | 新创建                                 | createAccount |
| new    | 新创建                                 | newAccount    |
| from   | 从既有的某物新建，或是从其他的数据新建 | fromConfig    |
| to     | 转换                                   | toString      |
| update | 更新既有某物                           | updateAccount |
| load   | 读取                                   | loadAccount   |
| fetch  | 远程读取                               | fetchAccount  |
| delete | 删除                                   | deleteAccount |
| remove | 删除                                   | removeAccount |
| save   | 保存                                   | saveAccount   |
| store  | 保存                                   | storeAccount  |
| commit | 保存                                   | commitChange  |
| apply  | 保存或应用                             | applyChange   |
| clear  | 清除数据或是恢复到初始状态             | clearAll      |
| reset  | 清除数据或是恢复到初始状态             | resetAll      |



##### 成对出现的动词

| 单词           | 意义              |
| :------------- | :---------------- |
| get获取        | set 设置          |
| add 增加       | remove 删除       |
| create 创建    | destory 移除      |
| start 启动     | stop 停止         |
| open 打开      | close 关闭        |
| read 读取      | write 写入        |
| load 载入      | save 保存         |
| create 创建    | destroy 销毁      |
| begin 开始     | end 结束          |
| backup 备份    | restore 恢复      |
| import 导入    | export 导出       |
| split 分割     | merge 合并        |
| inject 注入    | extract 提取      |
| attach 附着    | detach 脱离       |
| bind 绑定      | separate 分离     |
| view 查看      | browse 浏览       |
| edit 编辑      | modify 修改       |
| select 选取    | mark 标记         |
| copy 复制      | paste 粘贴        |
| undo 撤销      | redo 重做         |
| insert 插入    | delete 移除       |
| add 加入       | append 添加       |
| clean 清理     | clear 清除        |
| index 索引     | sort 排序         |
| find 查找      | search 搜索       |
| increase 增加  | decrease 减少     |
| play 播放      | pause 暂停        |
| launch 启动    | run 运行          |
| compile 编译   | execute 执行      |
| debug 调试     | trace 跟踪        |
| observe 观察   | listen 监听       |
| build 构建     | publish 发布      |
| input 输入     | output 输出       |
| encode 编码    | decode 解码       |
| encrypt 加密   | decrypt 解密      |
| compress 压缩  | decompress 解压缩 |
| pack 打包      | unpack 解包       |
| parse 解析     | emit 生成         |
| connect 连接   | disconnect 断开   |
| send 发送      | receive 接收      |
| download 下载  | upload 上传       |
| refresh 刷新   | synchronize 同步  |
| update 更新    | revert 复原       |
| lock 锁定      | unlock 解锁       |
| check out 签出 | check in 签入     |
| submit 提交    | commit 交付       |
| push 推        | pull 拉           |
| expand 展开    | collapse 折叠     |
| begin 起始     | end 结束          |
| start 开始     | finish 完成       |
| enter 进入     | exit 退出         |
| abort 放弃     | quit 离开         |
| obsolete 废弃  | depreciate 废旧   |
| collect 收集   | aggregate 聚集    |



## 函数规范

- 用函数名把判断、循环、递归等操作的含义显性化地表示，升代码的可读性和可理解性。
- 函数的第一规则是要短小，第二规则是要更短小。
- 一个方法只做一件事情，也就是函数级别的单一职责原则（`SRP`）。
- 减少辅助代码（判空、日志、鉴权等）对业务代码的干扰，直观地体现业务逻辑。
- 抽象层次一致性（Single Level of Abstraction Principle，SLAP），是和组合函数密切相关的一个原则。将一个大函数拆成多个子函数，函数体中的内容在同一个抽象层次。如果高层次抽象和底层细节杂糅在一起，就会难以理解。
- 使用函数式编程，减少冗余代码，让代码更简洁、可读性更好。函数是“无副作用”的，即没有对共享的可变数据操作，可以利用多核并行处理，而不用担心线程安全问题。
- 函数中添加中间变量来增加代码可读性。
- 避免重复的函数，把握函数功能的变与不变。



### Boolean值参数

向函数传入Boolean（书中称之为 Flag Argument）通常不是好主意。尤其是传入True or False后的行为并不是一件事情的两面，而是两件不同的事情时。这很明显违背了函数的单一职责约束，解决办法很简单，那就是用两个函数。



## 注释规范
- 注释不要为了复述代码功能，而是注释要解释代码背后的意图；



## 日志规范
比较有用的4个级别依次是ERROR、WARN、INFO和DEBUG。

### ERROR级别
ERROR表示不能自己恢复的错误，需要立即被关注和解决。例如，数据库操作错误、I/O错误（网络调用超时、文件读取错误等）、未知的系统错误（NullPointerException、OutOfMemoryError等）。

要做好ERROR输出的场景定义和规范，再配合监控治理，双管齐下，确保线上系统的稳定。



### WARN级别
对于可预知的业务问题，最好不要用ERROR输出日志，以免污染报警系统。例如，参数校验不通过、没有访问权限等业务异常，就不应该用ERROR输出。

需要注意的是，在短时间内产生过多的WARN日志，也是一种系统不健康的表现。



### INFO级别
INFO用于记录系统的基本运行过程和运行状态。INFO日志可初步定位，主要包括系统状态变化日志、业务流程的核心处理、关键动作和业务流程的状态变化。

切忌把INFO当成DEBUG使用，这样会导致记录的数据过多，一方面影响系统性能，日志文件增长过快，消耗不必要的存储资源；另一方面也不利于阅读日志文件。



### DEBUG级别
DEBUG是输出调试信息，如request/response的对象内容。



## 异常规范
### 异常处理
在业务系统中设定两个异常，分别是BizException（业务异常）和SysException（系统异常），而且这两个异常都应该是Unchecked Exception。业务异常使用WARN级别日志，系统异常使用ERROR级别日志。



### 错误码
#### 编号错误码
对于平台、底层系统或软件产品，可以采用编号式的编码规范，好处是编码风格固定，给人一种正式感；缺点是必须要配合文档才能理解错误码代表的意思。

淘宝开放平台也采用类似的编码方式，0~100表示平台解析错误，4表示User call limited（ISV调用次数超限）。

对不同的错误波段，一定要预留足够的码号。否则为了向后兼容，只能通过子错误码的方式进行变通处理。



#### 显性化错误码
显性化的错误码具有更强的灵活性，适合敏捷开发。

可以做一个约定：P代表参数异常（ParamException）、B代表业务异常（BizException）、S代表系统异常（SystemException）。



| 错误类型 | 错误码约定 | 举例                                        |
| -------- | ---------- | ------------------------------------------- |
| 参数异常 | P_XX_XX    | P_Customer_NameIsNull: 客户姓名不能为空     |
| 业务异常 | B_XX_XX    | B_Customer_NameAlreadyExist: 客户姓名已存在 |
| 系统异常 | S_XX_XX    | S_Unknow_Error：未知系统错误                |


# 设计原则
## SOLID原则
SOLID是5个设计原则开头字母的缩写，其本身就有“稳定的”的意思，寓意是“遵从SOLID原则可以建立稳定、灵活、健壮的系统”。



### 单一职责原则 Single Responsibility Principle（SRP）原则
SRP要求每个软件模块职责要单一，衡量标准是模块是否只有一个被修改的原因。职责越单一，被修改的原因就越少，模块的内聚性（Cohesion）就越高，被复用的可能性就越大，也更容易被理解。

每个类（类级别单一职责）或方法（方法级别单一职责）应该只有一个职责，对外只提供一种功能，引起类或方法变化的原因应该只有一个。

只有逻辑足够简单，才可以在类级别上违反单一职责原则；只有类中方法数量足够少，才可以在方法级别上违反单一职责原则；

单一职责原则优点：

1、降低复杂度；

2、提高可读性，提高系统可维护性；

3、降低变更引起的风险。



### 开闭原则 Open Close Principle（OCP）原则

开闭原则（Open Close Principle ）：对扩展开放，对修改关闭。

在程序需要进行拓展的时候，不能去修改原有的代码，实现一个热插拔的效果。所以一句话概括就是：为了使程序的扩展性好，易于维护和升级。想要达到这样的效果，我们需要使用接口和抽象类，后面的具体设计中我们会提到这点。

软件实体应该对扩展开放，对修改关闭。但是要注意可能会犯YAGNI（You Ain’t Gonna NeedIt）的错误。



### 里氏代换原则 Liskov Substitution Principle（LSP）原则

程序中的父类型都应该可以正确地被子类型替换。

里氏代换原则（Liskov Substitution Principle ）：任何父类可以出现的地方，子类一定可以出现。

里氏代换原则是继承复用的基石，只有当子类可以替换掉父类，软件单位的功能不受到影响时，父类才能真正被复用，而衍生类也能够在父类的基础上增加新的行为。里氏代换原则是对开闭原则的补充。实现开闭原则的关键步骤就是抽象化。而父类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。

 

当使用继承时，遵循里氏替换原则。类B继承类A时，除添加新的方法完成新增功能外，尽量不要重写父类A的方法，也尽量不要重载父类A的方法。

里氏替换原则通俗的来讲就是：子类可以扩展父类的功能，但不能改变父类原有的功能。它包含以下4层含义：

1、子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。

2、子类中可以增加自己特有的方法。

3、当子类的方法重载父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松。

4、当子类的方法实现父类的抽象方法时，方法的后置条件（即方法的返回值）要比父类更严格。



### 接口隔离原则 Interface Segregation Principle（ISP）原则

多个特定客户端接口要好于一个宽泛用途的接口。接口隔离原则认为不能强迫用户去依赖那些他们不使用的接口。换句话说，使用多个专门的接口比使用单一的总接口要好。

接口隔离原则（Interface Segregation Principle ）：一个接口不需要提供过多的行为，最好只提供一种对外的功能，不要把所有的操作都封装到一个接口中。

使用多个隔离的接口，比使用单个接口要好。降低类之间的耦合度，其实设计模式就是一个软件的设计思想，从大型软件架构出发，为了升级和维护方便。所以要降低依赖，降低耦合。



### 依赖倒置原则 Dependency Inversion Principle（DIP）原则

模块之间交互应该依赖抽象，而非实现。DIP要求高层模块不应该依赖于低层模块，二者都应该依赖于抽象。抽象不应该依赖细节，细节应该依赖抽象。

依赖倒置原则是（Dependence Inversion Principle ）：开闭原则的基础，具体内容：真对接口编程，依赖于抽象而不依赖于具体。



## DRY原则 
DRY是Don’t Repeat Yourself的缩写，DRY原则特指在程序设计和计算中避免重复代码，因为这样会降低代码的灵活性和简洁性，并且可能导致代码之间的矛盾。



## YAGNI原则
YAGNI是针对“大设计”（Big Design）提出来的，是“极限编程”提倡的原则，是指你自以为有用的功能，实际上都是用不到的。因此，除了核心的功能之外，其他的功能一概不要提前设计，这样可以大大加快开发进程。

DRY原则和YAGNI原则是不兼容的。前者追求“抽象化”，要求找到通用的解决方法；后者追求“快和省”，意味着不要把精力放在抽象化上面，因为很可能“你不会需要它”。因此，就有了Rule of Three原则。



## Rule of Three原则
三次原则指导我们可以通过以下步骤来写代码。
（1）第一次用到某个功能时，写一个特定的解决方法。
（2）第二次又用到的时候，复制上一次的代码。
（3）第三次出现的时候，才着手“抽象化”，写出通用的解决方法。



## KISS原则
把事情变复杂很简单，把事情变简单很复杂。好的目标不是越复杂越好，反而是越简洁越好。



## POLA原则原则
POLA（Principle of least astonishment）是最小惊奇原则，写代码不是写侦探小说，要的是简单易懂，而不是时不时冒出个“Surprise”。



# 编码相关

## 减少if-else

### 方法尽快返回结果

存在if-else结构时，考虑在if代码块中返回结果，减少if-else层级。

```java
public static void returnAsSoonAsYouCan(List<String> urls) {
    if(CollectionUtils.isEmpty(urls)) {
        // do business
        return;
    }
    // do business
}
```

### 使用枚举
根据不同的code返回不同的值时，可以使用枚举来减少if-else的使用。



### 使用Stream流
集合中的判断可以通过使用Stream流的方式来减少if-else的使用。



### 使用三元运算符
可以使用三元运算符来减少if-else的使用。



### 使用注解
通过自定义注解的方式来减少if-else的使用。

- 创建
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Pay {
    /**
     * 支付方式类型值
     *
     * @return aliPay, jingDongPay or weiXinPay
     */
    String value() default "";
}
```

```java
@Service
@Pay(PayConstant.ALI_PAY)
public class AliPayImpl implements IPayService {
    @Override
    public String pay() {
        return "支付宝支付";
    }
}
```

```java
@Service
@AllArgsConstructor
public class PayBizImpl implements IPayBiz, ApplicationListener<ContextRefreshedEvent> {
    private static Map<String, IPayService> INSTANCE_MAP = null;

    @Override
    public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
        ApplicationContext applicationContext = contextRefreshedEvent.getApplicationContext();
        Map<String, Object> beansWithAnnotation = applicationContext.getBeansWithAnnotation(Pay.class);

        if (MapUtils.isNotEmpty(beansWithAnnotation)) {
            INSTANCE_MAP = new HashMap<>(3);
            beansWithAnnotation.forEach((key, value) ->{
                String bizType = value.getClass().getAnnotation(Pay.class).value();
                INSTANCE_MAP.put(bizType, (IPayService) value);
            });
        }
    }

    @Override
    public String choosePay(String payType) {
        return INSTANCE_MAP.getOrDefault(payType, INSTANCE_MAP.get(PayConstant.ALI_PAY)).pay();
    }
}
```

# 一些误区
## 可复用性
可复用性不是目标，目标是软件的可维护性！不合适的复用会降低可维护性，有时越通用越无用。    

## OOP不是目的
OO只是解决问题的一种途径，也不是唯一的途径，千万不可把工具当目的。



---

