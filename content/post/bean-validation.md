---
title: "详解Bean Validation"
date: 2018-09-03T14:18:10+08:00
draft: true
---

来自 <https://my.oschina.net/u/3211616/blog/821343>:

## 1. Bean Validation是什么？

Bean Validation 是一个数据验证规范，属于Java EE 6的子规范，详情参考[维基百科](https://en.wikipedia.org/wiki/Bean_Validation)，这里不做赘述。

既然是规范，那么就会是实现者，目前最常用的也是最成熟的当属[hibernate-validator](http://hibernate.org/validator/)。

## 2. Bean Validation如何使用？

### eg 1：使用Bean Validation验证对象的属性
    
    // 1.有个POJO
    public class Person {
        // 2.使用Bean Validation constraint注解定义属性
        @NotBlank
        @Length(min = 1, max = 32)
        private String       name;
        @NotNull
        @Min(value = 1)
        private Integer      age;
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
        public Integer getAge() {
            return age;
        }
        public void setAge(Integer age) {
            this.age = age;
        }
    }
    public static void main(String[] args) {
        // 3.获取目标对象实例
        Person person = getTargetBeanInstance();
        // 4.获取bean validation验证类(单例即可)
        ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();
        Validator validator = validatorFactory.getValidator();
        // 5.验证对象属性
        Set<ConstraintViolation<Person>> set = validator.validate(person);
        // 6.输出验证结果
        for (ConstraintViolation<Person> constraintViolation : set) {
            System.out.println(constraintViolation.getMessage());
        }
    }
    

### eg 2：使用Bean Validation验证方法入参
    
    public interface TestService {
        // 1.使用Bean Validation constraint注解定义方法入参
        Person create(@NotNull @Valid Person person, @NotBlank String channel);
    }
    public static void main(String[] args) throws Exception {
        // 2.获取到目标类的实例
        TestService testService = getTargetService();
        // 3.获取目标类需要验证的方法
        Method method = testService.getClass().getMethod("create", Person.class, String.class);
        // 4.获取目标方法的入参
        Object[] parameterValues = { person, channel };
        // 5.获取bean validation验证类(单例即可)
        ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();
        Validator validator = validatorFactory.getValidator();
        // 6.验证目标方法的入参
        Set<ConstraintViolation<TestService>> set = validator.forExecutables().validateParameters(testService, method, parameterValues, new Class<?>[0]);
        // 7.输出验证结果
        for (ConstraintViolation<TestService> constraintViolation : set) {
            System.out.println(constraintViolation);
        }
    }
    

### eg 3：使用Bean Validation验证方法返回值
    
    public interface TestService {
        // 1.使用Bean Validation constraint注解定义方法返回值
        @NotNull @Valid Person create();
    }
    public static void main(String[] args) throws Exception {
        // 2.获取到目标类的实例
        TestService testService = getTargetService();
        // 3.获取目标类需要验证的方法
        Method method = testService.getClass().getMethod("create");
        // 4.获取目标方法的返回值
        Object returnValue = method.invoke(object);
        // 5.获取bean validation验证类(单例即可)
        ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();
        Validator validator = validatorFactory.getValidator();
        // 6.验证目标方法的返回值
        Set<ConstraintViolation<TestService>> set = validator.forExecutables().validateReturnValue(testService, method, returnValue, new Class<?>[0]);
        // 7.输出验证结果
        for (ConstraintViolation<TestService> constraintViolation : set) {
            System.out.println(constraintViolation);
        }
    }
    

**针对以上用法以及分析，简单做下总结：**

  * Bean Validation 可以验证对象属性、方法入参、方法返回值的合法性
  * Bean Validation 约束通过注解定义
  * 验证类：javax.validation.Validator
  * 验证结果输出：javax.validation.ConstraintViolation

## 3. Bean Validation工作机制

**通过上篇的示例，我们大概可以猜出Bean Validation的几个要点：**

  * constraint注解
  * constraint注解验证器
  * javax.validation.Validator实例化过程
  * javax.validation.Validator验证过程

**接下来也会以这几个要点来诠释Bean Validation工作机制**

### 3.1. Bean Validation工作机制 之 constraint注解

> **java里的注解是不能使用extends等关键字来定义继承关系的，所以constraint注解规范实际是在运行期间靠ConstraintHelper.isConstraintAnnotation方法做校验的，必须定义的属性有三个：message、groups、payload（有兴趣可以看下源码），姑且把所有constraint注解都必须定义的属性叫做“公共属性”，当前constraint注解个性化的属性叫做“私有属性”**

**分析javax.validation.constraints.Size的源码：**
    
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @Documented
    @Constraint(validatedBy = { }) // 公共：当前约束注解的验证器
    public @interface Size {
        // 私有属性：最小个数
        int min() default 0;
        // 私有属性：最大个数
        int max() default Integer.MAX_VALUE;
        // 公共属性：验证不通过的输出消息，支持占位符，例如："个数必须在{min}和{max}之间"
        String message() default "{javax.validation.constraints.Size.message}";
        // 公共属性：约束注解在验证时所属的组别
        Class<?>[] groups() default { };
        // 公共属性：约束注解的有效负载，目前没有处
        Class<? extends Payload>[] payload() default { };
        // 多约束，必须满足数组中的每个约束才算通过
        // 非必选，能用到的不多，但是每个constraint注解都定义了该注解，姑且当作公共吧
        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        @interface List {
            Size[] value();
        }
    }
    

### 3.2. Bean Validation工作机制 之 constraint注解验证器

**上面的例子中其实有提到验证器（注解@Constraint的validatedBy属性中定义的值），每个constraint注解都需要被特定的验证器执行**
    
    // 1.constraint注解验证器必须实现的接口：javax.validation.ConstraintValidator
    public class SizeValidatorForCollection implements ConstraintValidator<Size, Collection<?>> {
        private int min;
        private int max;
        @Override
        public void initialize(Size parameters) {
            // 2.初始化注解的私有属性，验证时会用到
            min = parameters.min();
            max = parameters.max();
        }
        @Override
        // 3.验证有效性
        public boolean isValid(Collection<?> collection, ConstraintValidatorContext constraintValidatorContext) {
            if ( collection == null ) {
                return true;
            }
            int length = collection.size();
            return length >= min && length <= max;
        }
    }
    

### 3.3. Bean Validation工作机制 之 javax.validation.Validator实例化过程

**观图方法：从Validation到Validator**

![image](http://github.aifaq.me/sharing/BeanValidation/analyze/images/BeanValidation.png)

**图看不明白？那就对照着文字逻辑看吧，如下：**

  1. javax.validation.Validation类是整个Bean Validation规范的入口类，它的主要职责是协助获取到ValidatorFactory实例，但是它并不负责创建ValidatorFactory实例，实际操作是委托给辅助类：GenericBootstrap或ProviderSpecificBootstrap

  2. GenericBootstrap类，帮助获取ValidationProvider 实例（实际操作是委托给ValidationProviderResolver类），它可以自定义ValidationProviderResolver，默认使用DefaultValidationProviderResolver

  3. ProviderSpecificBootstrap与GenericBootstrap的区别在于可以自定义ValidationProvider实现，所谓自定义，其实就是与目标对象做继承关系匹配

  4. ValidationProviderResolver类职责是帮助获取ValidationProvider实例，默认实现DefaultValidationProviderResolver是通过SPI方式获取ValidationProvider实例

  5. ValidationProvider创建Configuration，创建ValidatorFactory

  6. Configuration收集配置信息，也可创建ValidatorFactory实例（hibernate-validator实际委托职责给ValidationProvider）

  7. ValidatorFactory核心功能是创建Validator对象

  8. Validator类，整个Bean Validation初始化可以说为了最终得到该类实例，使用者使用它验证bean的有效性

**讲完javax.validation.Validator的实例化，那么接着就是Validator的验证过程，不过在这之前先讲两个概念：**

  * Bean Validation组
  * Bean Validation组序列

#### Bean Validation组

**Bean Validation组的概念是为了对约束分组，进而可以分组验证**

默认组：javax.validation.groups.Default

**注意：**

> 组也有继承的属性，对某组进行约束验证的时候，也会对其所继承的组(父类)进行验证。
    
    interface GroupA {}
    interface GroupA1 extends GroupA {}
    interface GroupB {}
    class Person {
        @NotBlank(groups = GroupA.class)
        private String       name;
    
        @NotNull(groups = {GroupA1.class, GroupB.class})
        private Integer      age;
    
        @NotBlank(groups = GroupB.class)
        private String       gender;
    }
    

  * 对GroupA验证，会验证name属性
  * 对GroupA1验证，会验证name、age属性
  * 对GroupB验证，会验证age、gender属性

#### Bean Validation组序列

**核心作用：定义组的验证顺序**
    
    interface GroupA {}
    interface GroupB {}
    interface GroupC {}
    @GroupSequence({GroupA.class, GroupB.class, GroupC.class}) 
    interface Group {} 
    class Person {
        @NotBlank(groups = GroupA.class)
        private String       name;
        @NotNull(groups = {GroupB.class})
        private Integer      age;
        @NotNull(groups = {GroupC.class})
        @Valid
        private Address       address;
    }
    

> **对Group验证，则会按照GroupA、GroupB、GroupC顺序验证**

**介绍完了Bean Validation组的概念后，我们接着看Validator的验证过程**

### 3.4. Bean Validation工作机制 之 javax.validation.Validator验证过程

**这里仅介绍使用Bean Validation验证对象属性的功能，其它验证方式都很相似，请自行阅读源码**

**核心方法：Set<ConstraintViolation<T>> validate(T object, Class<?>... groups)**

**判断逻辑主要有以下几步：**

  1. 判断目标Bean是否有约束，即是否使用了Bean Validation注解，判断的过程中会创建class与BeanMetaData的关系并缓存，下次直接读缓存

  2. 计算验证顺序，参考DefaultValidationOrder。  
先Group再Group的父类; 先Group再GroupSequence; 先当前Bean再级联Bean;

  3. 按顺序分组验证，大体逻辑就是通过反射获取到验证的值，使用@Constraint中定义的validatedBy(验证器，需要实现ConstraintValidator)来验证值的合法性

  4. 约束违规的消息解析，参考：AbstractMessageInterpolator  
消息解析有优先顺序，从高到底：  
ValidationMessages*.properties **>**  
ContributorValidationMessages*.properties **>** org.hibernate.validator.ValidationMessages*.properties

**到此基本介绍完了javax.validation.Validator的验证过程，那么我们应该对Bean Validation有了一个较完整的认识了，在此我们再次回顾下几个要点：**

  * constraint annotation：约束注解
  * constraint annotation validateBy：验证器
  * Bean Validation 组：分组、定义顺序
  * javax.validation.Validator：实例化过程
  * javax.validation.Validator：验证过程

**其实说了这么多，我们最终还是为了把Bean Validation使用到我们的项目中。**

## 4. Bean Validation如何做企业级应用？

**所谓企业级应用就是如何把这玩意应用到具体的项目中，我们主要以以下几点阐述这个问题：**

  * 如何定义自己的约束？
  * spring-mvc如何集成Bean Validation
  * 普通方法如何集成Bean Validation
  * dubbo如何集成Bean Validation

* * *

### 4.1. 如何自定义Bean Validation？

**大概有以下两步：**

#### 第一步：定义约束注解
    
    // 定义一个日期约束，必须是晚于指定日期
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @Documented
    @Constraint(validatedBy = { PastExtValidatorForDate.class, PastExtValidatorForCalendar.class })//公共：验证器，可以多个
    public @interface PastExt {
        // 私有：指定日期
        String value();
        // 公共：验证不通过的输出消息
        String message() default "{com.netease.validator.constraints.PastExt.message}";
        // 公共：
        Class<?>[] groups() default {};
        // 公共：
        Class<? extends Payload>[] payload() default {};
    
        @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
        @Retention(RUNTIME)
        @Documented
        @interface List {
            PastExt[] value();
        }
    }
    

#### 第二步：定义ConstraintValidator
    
    // 必须实现接口：ConstraintValidator，PastExt是约束注解，Date是约束的值类型
    public class PastExtValidatorForDate implements ConstraintValidator<PastExt, Date> {
    
        String value;
    
        @Override
        public void initialize(PastExt constraintAnnotation) {
            value = constraintAnnotation.value();
        }
        // 具体的约束验证逻辑
        @Override
        public boolean isValid(Date value, ConstraintValidatorContext context) {
            // null values are valid
            if (value == null) {
                return true;
            }
            return value.getTime() < DateHelper.parseDate(this.value).getTime();
        }
    }
    

* * *

### 4.2 spring-mvc如何集成Bean Validation

**spring-mvc集成Bean Validation是在Controller层，在参数解析的过程中做Bean的约束验证。前面我们说到的，Bean Validation验证类是javax.validation.Validator，那么该类实例是如何被创建的呢？**

> **spring并没有实现Bean Validation规范，只是做了层封装(你可以把她当做一个装饰者)，那么spring是如何创建javax.validation.Validator实例的呢？我们参考下：org.springframework.validation.beanvalidation.LocalValidatorFactoryBean（一般用这个）**

**这个验证类：org.springframework.validation.beanvalidation.LocalValidatorFactoryBean 主要功能就是创建javax.validation.Validator实例，并且使用org.springframework.context.MessageSource替代ValidationMessages*.properties（输出消息），有兴趣的可以看下源码**

**上面我们说到spring-mvc是在参数解析过程中做Bean的约束验证，那么有哪些解析器支持Bean Validation呢？**

**有两个：**

  * org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor
  * org.springframework.web.method.annotation.ModelAttributeMethodProcessor

**那么参数解析器是如何解析参数的呢？有兴趣的同学可以去研究下spring-mvc的参数解析器（很值得研究），上面的两个参数解析器主要解析使用以下几个注解修饰的参数：**

  * @RequestBody
  * @ResponseBody
  * @ModelAttribute

**我们再看下解析器中是如何做验证的：**

![image](http://github.aifaq.me/sharing/BeanValidation/analyze/images/resolveArgument.png) ![image](http://github.aifaq.me/sharing/BeanValidation/analyze/images/validateIfApplicable.png)

**可以看出：**

  * 方法入参必须要有 @Valid 或 @Validated 修饰才会做validate
  * 如果入参没有定义Errors，则抛出MethodArgumentNotValidException异常

**现在我们知道了spring-mvc是如何集成Bean Validation了，那么我们再看下在spring-mvc中如何使用Bean Validation？**

* * *

### 4.3. spring-mvc如何使用Bean Validation

**其实根据上篇文章可以得到，大概分两步：**

#### 第一步：使用Bean Validation约束注解定义请求参数

![image](http://github.aifaq.me/sharing/BeanValidation/analyze/images/test2.png)

#### 第二步：Bean Validation校验失败异常处理：@ControllerAdvice

![image](http://github.aifaq.me/sharing/BeanValidation/analyze/images/ExceptionHandler.png)

* * *

**上篇分析的是spring-mvc中请求参数的校验使用，但并不是所有的功能都是以web方式输出的，如果把Bean Validation集成到所有的普通方法中，会不会更通用**

### 4.4. 普通方法如何集成Bean Validation

> **思路：通过AOP拦截方法执行计划，做参数校验**

想偷懒有现成的吗？

**spring提供的方案：**
    
    <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor"/>;
    

#### MethodValidationPostProcessor实现原理

  * 拦截所有使用@Validated修饰的类（可以从父类继承注解），如果方法入参或者返回值被Bean Validation constraint注解修饰，则会做数据校验

**MethodValidationInterceptor如何校验方法返回值：**

![image](http://github.aifaq.me/sharing/BeanValidation/analyze/images/validateReturnValueMethod.png)

**可以看出校验失败会抛出ConstraintViolationException异常，所以，如果你的项目里需要使用MethodValidationPostProcessor，那么需要做四件事：**

  * spring xml 配置
  * 使用@Validated修饰你的目标类或其父类
  * 使用Bean Validation constraint注解修饰方法入参或者返回值
  * ConstraintViolationException异常处理

* * *

### 4.5 dubbo如何集成Bean Validation

dubbo支持JSR303标准注解验证，具体可参考其[官网](http://dubbo.io/User+Guide-zh.htm#UserGuide-zh-%E5%8F%82%E6%95%B0%E9%AA%8C%E8%AF%81)

**用法主要有两种，这里简单介绍下，不做详细描述：**

  * Consumer端开启Bean Validation
    
    <dubbo:reference id="xxxService" interface="x.xx.XXXService" validation="true" />
    

  * Provider开启Bean Validation
    
    <dubbo:service interface="x.xx.XXXService" ref="xxxService" validation="true" />
    

* * *

## 5. 为什么使用Bean Validation？

**官方slogan：Constrain once, validate everywhere**

> **使用Bean Validation最终的目的是为了统一规范，标准输出**

## 9. 常见问题

#### 9.1. 如何自定义错误消息？
    
    根据优先级，前面我们提到过，Bean Validation错误消息默认会读取三种文件，其优先级从高到底分别是：
    ValidationMessages*.properties
    ContributorValidationMessages*.properties
    org/hibernate/validator/ValidationMessages*.properties
    
    如果你用了spring，那么LocalValidatorFactoryBean会使用org.springframework.context.MessageSource替代ValidationMessages*.properties，所以使用spring的消息资源文件即可
    

#### 9.2. 嵌套的对象为什么不做验证？
    
    检查嵌套对象是否使用注解@Valid修饰
    

#### 9.3. 参数校验能否有一个不通过就返回，后续属性就不要再验证了？

**开启failFast**，如下：
    
    Configuration<?> configure = Validation.byDefaultProvider().configure();
    configure.addProperty(HibernateValidatorConfiguration.FAIL_FAST, "true");
    Validator validator = configure.buildValidatorFactory().getValidator();
    

