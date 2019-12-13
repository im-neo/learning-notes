# Java 反射动态获取和修改注解值

> 目的：通过注解 `TargetAnnotation` 的配置，动态获取和修改注解 `Property` 值

## 创建自定义注解：`TargetAnnotation.java`
```java
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Documented
@Target(ElementType.FIELD)
@Retention(value = RetentionPolicy.RUNTIME)
public @interface TargetAnnotation {
    /**
     * 目标注解类
     */
    Class<?> targetAnnotationClass() default Void.class;

    /**
     * 目标注解属性
     */
    String targetAnnotationClassField() default "";
}
```

## 创建自定义注解：`Property.java`
```java
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Documented
@Target(ElementType.FIELD)
@Retention(value = RetentionPolicy.RUNTIME)
public @interface Property {

    String name() default "";
}
```

## 创建测试Bean：`Bean.java`
```java
public class Bean {
    
    @Property(name = "名称")
    @TargetAnnotation(targetAnnotationClass = Property.class , targetAnnotationClassField = "name")
    private String name;
}
```

## 测试
```java
import java.lang.annotation.Annotation;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;
import java.util.Map;

public class Test {

    public static void main(String[] args) throws Exception {
        Field nameField = Bean.class.getDeclaredField("name");

        TargetAnnotation targetAnnotation = nameField.getAnnotation(TargetAnnotation.class);

        // 获取标注的目标注解类
        Class<? extends Annotation> targetAnnotationClass = (Class<? extends Annotation>) targetAnnotation.targetAnnotationClass();

        // 获取 TargetAnnotation.targetAnnotationClass 标注的注解代理实例
        Annotation annotation = nameField.getAnnotation(targetAnnotationClass);

        // 获取代理实例所持有的 InvocationHandler
        InvocationHandler invocationHandler = Proxy.getInvocationHandler(annotation);

        // 获取 AnnotationInvocationHandler 的 memberValues 字段
        Field memberValuesField = invocationHandler.getClass().getDeclaredField("memberValues");

        // 打开访问权限
        memberValuesField.setAccessible(true);

        // 获取 memberValues (目标注解的信息都在 memberValues 中)
        Map memberValues = (Map) memberValuesField.get(invocationHandler);

        // 1)、动态获取 TargetAnnotation.targetAnnotationClassField 标注的注解属性值
        Object targetAnnotationClassField = memberValues.get(targetAnnotation.targetAnnotationClassField());
        System.out.println("修改前：" + targetAnnotationClassField);

        // 2)、动态修改 TargetAnnotation.targetAnnotationClassField 标注的注解属性值
        memberValues.put(targetAnnotation.targetAnnotationClassField(), "用户名");

        // 3)、再次获取修改后的 TargetAnnotation.targetAnnotationClassField 标注的注解属性值
        targetAnnotationClassField = memberValues.get(targetAnnotation.targetAnnotationClassField());
        System.out.println("修改后：" + targetAnnotationClassField);
    }
}
```

## 测试结果
```
修改前：名称
修改后：用户名
```