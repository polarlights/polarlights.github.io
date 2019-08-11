---
title: bean validation tutorial
categories:
  - java
tags:
  - bean validation
date: 2019-08-03 15:50:02
---

## Preface
在开发 Web 接口的时候，客户端给到的输入参数往往是不可控，亦不可信的；需要在 server 端做一些校验，以使处理数据符合业务预期；除了web 接口之外，我们写的方法、DAO 对象都有类似的需求。也就是校验的需求贯穿服务的各个层。

简单粗暴一点的可以在每一层这样写：
```
if (param.getName() == null) {
  throw BadRequestException("name can not be null");
}
if (param.getAge() < 1) {
  throw BadRequestException("age can not less than 1");
}
```

如果遇到错误，我想最后统一抛出，用户可以一次知道哪些输入是有问题。那么我们可以这样写:

```
List<String> errorMessages = new ArraysList<>();

if (params.getName() == null) {
  errorMessages.add("name can not be null");
}

if (param.getAge() < 1) {
  errorMessages.add("age can not less than 1");
}

if (errorMessages.size() > 0) {
  ...
}
```

这样写会有什么样的问题呢？

① 通用性不够
② 重复代码很多

那么如何解决这个问题呢？ Java 提供了 Bean Validation 的概念，先后经过了 [[JSR303, 2009]](https://beanvalidation.org/1.0/spec/)、[[JSR 349, 2013]](https://beanvalidation.org/1.1/spec/), [[JSR 380, 2017]](https://beanvalidation.org/2.0/spec/) 三个标准。 Bean Validation 的目的是解决程序从表现层到持久层的对象的重复校验逻辑。它使用注解、XML 对对象的属性进行约束。

除了默认提供的校验注解，Bean Validation 还提供了自定义注解来实现自己的校验逻辑、本地化违反约定信息等能力。

下文我们主要以 Bean Validation 2.0为例进行讲解。

## 使用

Bean Validation 2.0的官方认证实现为[Hibernate Validator](https://hibernate.org/validator/documentation/)。spring boot 项目只要引入了 spring-boot-starter-web 依赖就自动引入了相关依赖。

它需要的依赖有:

1. javax.validation:validation-api
2. org.hibernate.validator:hibernate-validator

前者提供抽象描述和抽象接口，后者提供具体实现。

## 常用约束

注解 | 适用对象类型 | 说明 | null 是否被视为有效
--- | --- | --- | ---
AssertFalse | boolean | 元素必须为 false；| Y
AssertTrue | boolean | 元素必须为 true； | Y
DecimalMax | BigDecimal, BigInteger, CharSequence, byte/short/int/long 及包装类型| 元素必须小于等于给定值；| Y
DecimalMin | BigDecimal, BigInteger, CharSequence, byte/short/int/long 及包装类型| 元素必须大于等于给定值；| Y
Digits | BigDecimal, BigInteger, CharSequence, byte/short/int/long 及包装类型| 元素的整数与分数部分分别约束最大值；| Y
Email | String | 字符串必须是有效的邮件地址 | N
Future | Date/Calendar/Instant/LocalDate/LocalTime/LocalDateTime/MonthDay/OffsetDateTime/OffsetTime/Year/YearMonth/ZonedDateTime |  元素必须大于当前时间 | Y
FutureOrPresent | Date/Calendar/Instant/LocalDate/LocalTime/LocalDateTime/MonthDay/OffsetDateTime/OffsetTime/Year/YearMonth/ZonedDateTime |  元素必须大于等于当前时间 | Y
Max | BigDecimal/BigInteger/byte/short/int/long | 元素小于等于给定值 | Y
Min | BigDecimal/BigInteger/byte/short/int/long | 元素大于等于给定值 | Y
NotEmpty | String/Collection/Map/Array | 元素不能为 null 或者为空 | N
NotNull | any | 元素不能为 null | N
Post | Date/Calendar/Instant/LocalDate/LocalTime/LocalDateTime/MonthDay/OffsetDateTime/OffsetTime/Year/YearMonth/ZonedDateTime |  元素必须小于当前时间 | Y
Pattern | String | 字符串必须符合给定的正则 | Y
Size | String/Collection/Map/Array | 元素的长度/元素数量在给定范围内 | Y

## 简单使用

在 Controller 参数前添加 @Valid 注解，即:
```java
// model
public class UserRequest {
  @NotEmpty
  private String name;
  @Range(min=1, max=125)
  private int age;
  @Valid
  private Address address;
}

# controller
@PostMapping("/users")
public String createUser(@Valid @RequestBody UserRequest user);
```

**需要注意的是：**① 如果对象嵌套了其它对象(即需要级联校验)，需要在里面使用 @Valid 注解 ② 如果 controller 使用继承实现，那么要符合里氏替换原则原则（即子类的约束被强化或者弱化; 父方法的约束会自然被子类方法继承)。

另外 Spring 默认不提供方法级别的校验，如果需要校验，需要在类级别添加`@Validated`注解。

如果想全局处理 Bean Validation 异常，则可以再 ErrorAdvice 处理类中，添加对`MethodArgumentNotValidException`异常处理即可。如果添加了对方法参数的校验，还需要再 ErrroAdvice 中添加 `ConstraintViolationException`异常类的处理。

## 更多用法

### 自定义约束

如果默认的校验不能满足业务场景的需要，我们可以自定义约束。

每个约束注解的定义包括但不限于：

1. 注解的校验类 [非必需]
2. 校验不通过的 message 字符串 [必需]
3. 注解所属的分组(Group) [必需]
4. 注解所属的负载(Payload) [必需]

假如说现在有一个场景，某个字符串属性，只接受特定的几个值.
```java
// 定义约束注解
@Documented
@Constraint(validatedBy = {StringRangeValidator.class})
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
@Retention(RUNTIME)
public @interface StringRange {

    String[] values() default {};

    String message() default "属性值只能在指定的范围内";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}

// 定义校验实现类
public class StringRangeValidator implements ConstraintValidator<StringRange, String> {

    Set<String> valueSets = new HashSet<>();

    @Override
    public void initialize(StringRange constraintAnnotation) {
        for (String str : constraintAnnotation.values()) {
            if (Objects.nonNull(str)) {
                valueSets.add(str);
            }
        }
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        return valueSets.contains(value);
    }
}
```

这样我们就很方便地实现了一个自定义的校验类。

### 分组校验

分组校验主要用于同一个 Bean 对象的特定属性在不同场景下约束行为不同服务的。比如一个 User 对象，创建与更新的对象大部分是相同的，不同的是更新时 ID 不为空，而创建时必须为空。那么我们可以分别定义两个组(组名一般是接口; 默认分组是 javax.validation.groups.Default.class)：

```java
public interface CreatedGroup {}
public interface UpdatedGroup {}

// User.java

@NotNull(group = UpdatedGroup.class)
@Null(group = CreatedGroup.class)
private Long id;
```

在启用校验的地方:
```java
@RequestMapping("/users")
public User createUser(@Validated(CreatedGroup.class) CreateUserRequest createUserRequest) { ... }
```

## 其它

我们没有处理的有 Payload(负载)的使用，它是附加在 Group 之外的一种元数据描述，一种用途是：描述校验的错误严重级别。因为使用较少，我们不多描述。有兴趣的朋友可以读一下Bean Validation 的 spec 文档。

如果我们在没有 Spring 的条件下想使用 Bean Validation 怎么办？
```java
Validator validator = Validation.buildDefaultValidatorFactory().getValidator();

validator.validate(instantce, constraintGroups);
```

关联 repo: https://github.com/polarlights/bean_validation_tutorial

## 参考资料

1. [使用Bean Validation实现数据校验](https://www.jianshu.com/p/2a495bf5504e)
2. [JSR 303 - Bean Validation 介绍及最佳实践](https://www.ibm.com/developerworks/cn/java/j-lo-jsr303/index.html)
3. [Bean validation specification](https://beanvalidation.org/2.0/spec/#introduction)
7. [Bean validation payload example](https://www.logicbig.com/how-to/code-snippets/jcode-bean-validation-javax-validation-payload.html)
8. https://www.logicbig.com/tutorials/java-ee-tutorial/bean-validation.html
9. [Hibernate Validator Documentation](https://hibernate.org/validator/documentation/)