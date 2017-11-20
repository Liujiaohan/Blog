# 一、简述
##  1.什么是AOP
AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
## 2.什么时候使用AOP
举个例子：当我们的项目有多个入口时，访问不同的页面都要做用户验证，在不使用AOP时我们可以调用一个专门的方法，来完成需求。但如果很多地方都要求呢？把重复代码再写一遍？后期项目需求又改了？作为一个有追求的程序员这样太不优雅了。不就是在方法前插入一段代码吗？AOP分分钟搞定。除此之外，AOP还可以用于记录日志，计算方法执行时间等等

## 3.概念
1. 通知、增强处理（Advice）：就是你想要的功能，也就是上面说的日志、耗时计算等。
2. 连接点（JoinPoint）：允许你通知（Advice）的地方，那可就真多了，基本每个方法的前、后（两者都有也行），或抛出异常是时都可以是连接点（spring只支持方法连接点）。AspectJ还可以让你在构造器或属性注入时都行，不过一般情况下不会这么做，只要记住，和方法有关的前前后后都是连接点。
3. 切入点（Pointcut）：上面说的连接点的基础上，来定义切入点，你的一个类里，有15个方法，那就有十几个连接点了对吧，但是你并不想在所有方法附件都使用通知（使用叫织入，下面再说），你只是想让其中几个，在调用这几个方法之前、之后或者抛出异常时干点什么，那么就用切入点来定义这几个方法，让切点来筛选连接点，选中那几个你想要的方法。
4. 切面（Aspect）：切面是通知和切入点的结合。现在发现了吧，没连接点什么事，连接点就是为了让你好理解切点搞出来的，明白这个概念就行了。通知说明了干什么和什么时候干（什么时候通过before，after，around等AOP注解就能知道），而切入点说明了在哪干（指定到底是哪个方法），这就是一个完整的切面定义。
5. 织入（weaving） 把切面应用到目标对象来创建新的代理对象的过程。
6. 目标对象 – 项目原始的Java组件。
7. AOP代理  – 由AOP框架生成java对象。
8. AOP代理方法 = advice +　目标对象的方法。

## 二.AOP实现方式
### 1.AspectJ
AspectJ: 一个 JavaTM 语言的面向切面编程的无缝扩展（适用Android）

### 2.AOP注解与使用
@Aspect：声明切面，标记类  
@Pointcut(切点表达式)：定义切点，标记方法  
@Before(切点表达式)：前置通知，切点之前执行  
@Around(切点表达式)：环绕通知，切点前后执行  
@After(切点表达式)：后置通知，切点之后执行  
@AfterReturning(切点表达式)：返回通知，切点方法返回结果之后执行  
@AfterThrowing(切点表达式)：异常通知，切点抛出异常时执行  
1.  切点表达式是什么？
```
execution(<修饰符模式>? <返回类型模式> <方法名模式>(<参数模式>) <异常模式>?)
举个例子
@Before("execution(public * *(..))")
public void before(JoinPoint point) {
    System.out.println("Before");
}

@AfterThrowing(value = "execution(* com.lqr..*(..))", throwing = "ex")
public void afterThrowing(Throwable ex) {
    System.out.println("ex = " + ex.getMessage());
}
```
2. @PointCount的使用
```
@Pointcut是专门用来定义切点的，让切点表达式可以复用。
举个例子

@Pointcut("execution(public * *(..))")
public void pointcut() {}

@Before("pointcut")
public void before(JoinPoint point) {
    System.out.println("Before");
}
```
## 三、实战
计算方法耗时
### 切面
```
@org.aspectj.lang.annotation.Aspect
public class Aspect {
    @Pointcut("execution(public * *(..))")
    public void pointcut(){}

    @Around("pointcut()")
    public void timer(ProceedingJoinPoint proceedingJoinPoint)throws Throwable{
        System.out.println("@Around");
        long beginTime= System.currentTimeMillis();
        proceedingJoinPoint.proceed();
        long endTime=System.currentTimeMillis();
        long dx=endTime-beginTime;
        System.out.println("耗时："+dx+"ms");
    }

    @Before("pointcut()")
    public void before(JoinPoint point) {

        System.out.println("@Before"+point.getTarget());
    }

    @After("pointcut()")
    public void after(JoinPoint point) {
        System.out.println("@After");
    }

    @AfterReturning("pointcut()")
    public void afterReturning(JoinPoint point, Object returnValue) {
        System.out.println("@AfterReturning");
    }

    @AfterThrowing(value = "pointcut()", throwing = "ex")
    public void afterThrowing(Throwable ex) {
        System.out.println("@afterThrowing");
        System.out.println("ex = " + ex.getMessage());
    }
}
```

### 结果
```
@Around
@Beforenull
@Around
@Beforenull
I am test()
耗时：2ms
@After
@AfterReturning
耗时：4ms
@After
@AfterReturning
```
## 注解切点
切点表达式的完整结构其实应该是这样的
execution(<@注解类型模式>? <修饰符模式>? <返回类型模式> <方法名模式>(<参数模式>) <异常模式>?)
1. 自定义注解
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AnnoJoinPoint {
    String value();
    int type();
}
```
2. 修改切点
```
    @Pointcut("execution(@com.bingyanstudio.wireless.aop.AnnoJoinPoint public * *(..))")
    public void pointcut(){}
```
3. 结果
```
@Around
@Beforenull
I am test()耗时：1ms
@After
@AfterReturning
```

3. 获取注解参数
```
MethodSignature signature = (MethodSignature) joinPoint.getSignature();
Method method = signature.getMethod();
// 通过Method对象得到切点上的注解
AnnoJoinPoint annotation = method.getAnnotation(AnnoJoinPoint.class);
String value = annotation.value();
int type = annotation.type();
```



