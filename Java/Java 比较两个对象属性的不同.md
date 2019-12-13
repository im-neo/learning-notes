# Java 比较两个对象属性的不同
> 已实现以下需求：
> 1. 相同对象不同对象都可比较
> 2. 对象和 Map 可比较，Map 和 Map 可比较
> 3. 可以自定义比较的方式（如：Date 只比较年月日，不比较时分秒），可指定全局处理方式 ，可指定特定字段的处理方式，通过实现：`TypeProcessHandle.isDifferent(Object o1, Object o2)`
> 4. 可自定义返回结果中属性值的格式化方式（如：1.数据字典值和文本的转换 2.id 或 code 和对应名称的转换）通过实现：`TypeProcessHandle.format(Class<?> clazz , String fieldName , Object value)`
> 5. 可根据配置在结果中返回字段的说明文字
> 6. 可指定返回结果中 JSON 的 Key
> 7. 可指定需要比较的字段
> 
> 可用于生产中需要记录修改记录的场景
> 
> 如有其它需求或建议，请评论！

## 包依赖
```xml
<dependency>
     <groupId>com.google.guava</groupId>
     <artifactId>guava</artifactId>
     <version>28.0-jre</version>
</dependency>
<dependency>
     <groupId>com.alibaba</groupId>
     <artifactId>fastjson</artifactId>
     <version>1.2.62</version>
</dependency>
<dependency>
    <groupId>org.reflections</groupId>
    <artifactId>reflections</artifactId>
    <version>0.9.11</version>
</dependency>
```


## 使用说明
### 方法说明
| 方法名    | 说明 |
| :---      | :--- |
| `compareFields(String... compareFields)` | 指定需要比较的字段，不指定即为所有字段，Map 除外 |
| `ignoreCompareFields(String... ignoreCompareFields)` | 指定需要忽略比较的字段，也可通过：`Property.isIgnore()` 指定 |
| `fieldTitleKey(String fieldTitleKey)` | 指定属性标题，默认：title |
| `fieldNameKey(String fieldNameKey)`| 指定属性名，默认：key |
| `newValueKey(String newValueKey)`| 指定新值Key，默认：after |
| `oldValueKey(String oldValueKey)`| 指定老值Key，默认：before |
| `isBuilderNullValue(boolean isBuilderNullValue)` | 指定是否空值是否返回，默认：false |
| `putFieldTitleMapping(String field, String title)` | 指定属性名和标题，也可用`Property.name()` 指定 |
| `configGlobalNameAnnotation(Class<? extends Annotation> annotation , String field)` | 配置全局标题所在的注解属性，如果需要单独配置需要使用 `Property.nameAnnotationClass()` 和 `Property.nameAnnotationClassField()` 注解指定 |
| `registerProcessHandle(TypeProcessHandle handle)` | 注册处理器 |
| `putFieldNameProcessHandleMapping(String field, TypeProcessHandle handle)` | 指定属性处理器，也可通过： `Property.typeProcessHandleUsing()` 指定 |

### 优先级说明
- 处理器优先级
> `BuilderDifferenceInfoHandle.Builder#putFieldNameProcessHandleMapping()` > `Property#typeProcessHandleUsing()` > `BuilderDifferenceInfoHandle.Builder#registerProcessHandle()` > 内置处理器(`INNER_TYPE_PROCESS_HANDLE_MAP`) > 默认

- 属性名称优先级
> `BuilderDifferenceInfoHandle.Builder#putFieldTitleMapping()` > `Property#nameAnnotationClass() > Property#name()` > `BuilderDifferenceInfoHandle.Builder#configGlobalNameAnnotation()`

### 自定义内置处理器说明
> - 需要实现 `TypeProcessHandle` 接口
> - 需要在自定义内置处理器上标注 `InnerTypeProcessHandle` 注解
> - 需要和 `BuilderDifferenceInfoHandle` 处于同一个包
> 
> 以上三点需要同时满足


## 定义类型处理接口：`TypeProcessHandle.java`
```java
/**
 * 类型处理器
 *
 * @Author: Neo
 * @Date: 2019/11/22 23:13
 * @Version: 1.0
 */
public interface TypeProcessHandle<T> {
    
    /**
     * 判断两个值是否不同
     *
     * @Author: Neo
     * @Date: 2019/11/22 23:13
     * @Version: 1.0
     */
    boolean isDifferent(Object o1, Object o2);

    /**
     * 支持的类型
     *
     * @Author: Neo
     * @Date: 2019/11/22 23:14
     * @Version: 1.0
     */
    Class<?> supportTypeKey();
    
    /**
     * 格式化
     * @param clazz 比较的对象
     * @param fieldName 属性
     * @param value 属性值
     *
     * @Author: Neo
     * @Date: 2019/11/23 0:13
     * @Version: 1.0
     */
    Object format(Class<?> clazz , String fieldName , Object value);
}
```

## 定义三个类型处理实现类
- `BigDecimalTypeProcessHandle.java`

```java
import com.alibaba.fastjson.util.TypeUtils;

import java.math.BigDecimal;
import java.util.Objects;

/**
 * 时间类型处理器
 *
 * @Author: Neo
 * @Date: 2019/11/24 17:40
 * @Version: 1.0
 */
@InnerTypeProcessHandle
public class BigDecimalTypeProcessHandle implements TypeProcessHandle<BigDecimal>{
    @Override
    public boolean isDifferent(Object o1, Object o2) {
        BigDecimal d1 = TypeUtils.castToBigDecimal(o1);
        BigDecimal d2 = TypeUtils.castToBigDecimal(o2);
        return Objects.isNull(d1) ? Objects.isNull(d2) : d1.compareTo(d2) != 0;
    }

    @Override
    public Class<?> supportTypeKey() {
        return BigDecimal.class;
    }

    @Override
    public Object format(Class<?> clazz, String fieldName, Object value) {
        return value;
    }
}
```

- `DateTypeProcessHandle.java`

```java
import com.alibaba.fastjson.util.TypeUtils;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Objects;

/**
 * 时间类型处理器
 *
 * @Author: Neo
 * @Date: 2019/11/24 17:40
 * @Version: 1.0
 */
@InnerTypeProcessHandle
public class DateTypeProcessHandle implements TypeProcessHandle<Date> {

    public static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    @Override
    public boolean isDifferent(Object o1, Object o2) {
        Date d1 = TypeUtils.castToDate(o1);
        Date d2 = TypeUtils.castToDate(o2);

        return Objects.isNull(d1) ? Objects.isNull(d2) : d1.compareTo(d2) != 0;
    }

    @Override
    public Class<?> supportTypeKey() {
        return Date.class;
    }


    @Override
    public Object format(Class<?> clazz, String fieldName, Object value) {
        Date date = TypeUtils.castToDate(value);
        if (Objects.isNull(date)) {
            return null;
        }
        return sdf.format(date);
    }
}
```

- `DefaultTypeProcessHandle.java`

```java
import com.alibaba.fastjson.util.TypeUtils;
import org.apache.commons.lang3.StringUtils;


/**
 * 默认类型处理器
 *
 * @Author: Neo
 * @Date: 2019/11/24 17:40
 * @Version: 1.0
 */
public class DefaultTypeProcessHandle implements TypeProcessHandle<Object> {
    @Override
    public boolean isDifferent(Object o1, Object o2) {
        String n = TypeUtils.castToString(o1);
        String o = TypeUtils.castToString(o2);
        return !StringUtils.equals(n, o);
    }

    @Override
    public Class<?> supportTypeKey() {
        return Object.class;
    }

    @Override
    public Object format(Class<?> clazz, String fieldName, Object value) {
        return TypeUtils.castToString(value);
    }
}
```

## 定义必要的注解类：
- `Property.java`

```java
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;


/**
 * 组装修改信息属性注解
 *
 * @Author: Neo
 * @Date: 2019/11/23 11:24
 * @Version: 1.0
 */
@Documented
@Target(ElementType.FIELD)
@Retention(value = RetentionPolicy.RUNTIME)
public @interface Property {
    /**
     * 名称
     */
    String name() default "";

    /**
     * 类型处理使用类
     */
    Class<?> typeProcessHandleUsing() default Void.class;

    /**
     * 名称标注注解
     * 单独使用无效，需配合{@link #nameAnnotationClassField}
     */
    Class<?> nameAnnotationClass() default Void.class;

    /**
     * 名称标注注解属性
     * 单独使用无效，需配合{@link #nameAnnotationClass}
     */
    String nameAnnotationClassField() default "";

    /**
     * 是否忽略改属性
     */
    boolean isIgnore() default false;
}
```

- `InnerTypeProcessHandle.java` 

```java
/**
 * 内置处理器标志
 * 标注在 {@link TypeProcessHandle} 的实现类上才有效
 * 
 * @Author: Neo
 * @Date: 2019/12/13 21:38
 * @Version: 1.0
 */
@Documented
@Target(ElementType.TYPE)
@Retention(value = RetentionPolicy.RUNTIME)
public @interface InnerTypeProcessHandle {}
```

## 使用两个其它工具类
- `ArrayUtils.java`

```java
import java.util.Objects;

/**
 * 数组工具
 *
 * @Author: Neo
 * @Date: 2019/11/24 16:49
 * @Version: 1.0
 */
public class ArrayUtils {

    private ArrayUtils() {
    }

    /**
     * 数组长度
     *
     * @Author: Neo
     * @Date: 2019/11/24 16:49
     * @Version: 1.0
     */
    public static int length(Object[] array) {
        return Objects.isNull(array) ? 0 : array.length;
    }

    /**
     * 是否为空数组
     *
     * @Author: Neo
     * @Date: 2019/11/24 17:02
     * @Version: 1.0
     */
    public static boolean isEmpty(Object[] array) {
        return length(array) == 0;
    }

    /**
     * 不否不为空数组
     *
     * @Author: Neo
     * @Date: 2019/11/24 17:02
     * @Version: 1.0
     */
    public static boolean isNotEmpty(Object[] array) {
        return !isEmpty(array);
    }
}
```

- `ReflectUtils.java`(非原创)

```java
import org.apache.commons.lang3.ArrayUtils;

import java.lang.annotation.Annotation;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

/**
 * 类反射操作工具类
 *
 * @author dujunjun
 * @date 2014年11月20日 下午3:27:17
 */
public class ReflectUtils {


    /**
     * 根据字段名称获取对象字段信息
     *
     * @param obj       对象
     * @param fieldName 字段名称
     * @return
     */
    public static Field getFieldByFieldName(Object obj, String fieldName) {
        return getFieldByFieldName(obj.getClass(), fieldName);
    }

    /**
     * 根据字段名称获取class字段信息
     *
     * @param clazz     class
     * @param fieldName 字段名称
     * @return
     */
    public static Field getFieldByFieldName(Class<?> clazz, String fieldName) {
        for (Class<?> superClass = clazz; superClass != Object.class; superClass = superClass.getSuperclass()) {
            try {
                return superClass.getDeclaredField(fieldName);
            } catch (NoSuchFieldException e) {
            }
        }
        return null;
    }

    /**
     * 获取对象字段值
     *
     * @param obj       对象
     * @param fieldName 字段
     * @return
     * @throws NoSuchFieldException
     * @throws IllegalAccessException
     */
    public static Object getValueByFieldName(Object obj, String fieldName)
            throws IllegalAccessException {
        // 是map
        if (Map.class.isAssignableFrom(obj.getClass())) {
            Map map = (Map) obj;
            return map.get(fieldName);
        }
        Field field = getFieldByFieldName(obj, fieldName);
        Object value = null;
        if (field != null) {
            if (field.isAccessible()) {
                value = field.get(obj);
            } else {
                field.setAccessible(true);
                value = field.get(obj);
                field.setAccessible(false);
            }
        }
        return value;
    }

    /**
     * 设置对象字段值
     *
     * @param obj       对象
     * @param fieldName 字段
     * @param value
     * @throws NoSuchFieldException
     * @throws IllegalAccessException
     */
    public static void setValueByFieldName(Object obj, String fieldName,
                                           Object value) throws NoSuchFieldException, IllegalAccessException {
        Field field = obj.getClass().getDeclaredField(fieldName);
        if (field.isAccessible()) {
            field.set(obj, value);
        } else {
            field.setAccessible(true);
            field.set(obj, value);
            field.setAccessible(false);
        }
    }

    /***
     * 获取某一个实体的数据类型
     *
     * @param clazz
     *            实体字节码文件
     * @param fieldName
     *            字段名称
     * @return Class<?>
     * @throws NoSuchFieldException
     */
    public static Class<?> findParameterClass(Class<?> clazz, String fieldName)
            throws NoSuchFieldException {
        try {
            return clazz.getDeclaredField(fieldName).getType();
        } catch (Exception e) {
            Class<?> sup = clazz.getSuperclass();
            return sup.getDeclaredField(fieldName).getType();
        }
    }

    /**
     * 获得class中所有的field，包括继承父类的field
     *
     * @param clazz
     * @return
     */
    public static Field[] getAllFields(Class<?> clazz) {
        Field[] fields = null;
        for (; clazz != Object.class; clazz = clazz.getSuperclass()) {
            fields = ArrayUtils.addAll(fields, clazz.getDeclaredFields());
        }
        return fields;
    }

    /**
     * 获得本类中有<code>annotationClazz</code>注解的字段数组
     *
     * @param clazz
     * @param annotationClazz
     * @return
     */
    public static Field[] getAnnotationFields(Class<?> clazz, Class<? extends Annotation> annotationClazz) {
        Field[] fields = null;
        Field[] allFields = getAllFields(clazz);
        for (int i = 0; i < allFields.length; i++) {
            Field f = allFields[i];
            if (f.isAnnotationPresent(annotationClazz)) {
                fields = ArrayUtils.add(fields, f);
            }
        }
        return fields;
    }

    /**
     * 得到指定类型的指定位置的泛型实参
     *
     * @param clazz
     * @param index
     * @param <T>
     * @return
     */
    @SuppressWarnings("unchecked")
    public static <T> Class<T> findParameterizedType(Class<?> clazz, int index) {
        Type parameterizedType = clazz.getGenericSuperclass();
        // CGLUB subclass target object(泛型在父类上)
        if (!(parameterizedType instanceof ParameterizedType)) {
            parameterizedType = clazz.getSuperclass().getGenericSuperclass();
        }
        if (!(parameterizedType instanceof ParameterizedType)) {
            return null;
        }
        Type[] actualTypeArguments = ((ParameterizedType) parameterizedType)
                .getActualTypeArguments();
        if (actualTypeArguments == null || actualTypeArguments.length == 0) {
            return null;
        }
        return (Class<T>) actualTypeArguments[0];
    }

    /**
     * 通过反射,获得指定类的父类的泛型参数的实际类型. 如BuyerServiceBean extends DaoSupport<Buyer>
     *
     * @param clazz clazz 需要反射的类,该类必须继承范型父类
     * @param index 泛型参数所在索引,从0开始.
     * @return 范型参数的实际类型, 如果没有实现ParameterizedType接口，即不支持泛型，所以直接返回
     * <code>Object.class</code>
     */
    public static Class<?> getSuperClassGenericType(Class<?> clazz, int index) {
        // 得到泛型父类
        Type genType = clazz.getGenericSuperclass();
        // 如果没有实现ParameterizedType接口，即不支持泛型，直接返回Object.class
        if (!(genType instanceof ParameterizedType)) {
            return Object.class;
        }
        // 返回表示此类型实际类型参数的Type对象的数组,数组里放的都是对应类型的Class, 如BuyerServiceBean extends
        // DaoSupport<Buyer,Contact>就返回Buyer和Contact类型
        Type[] params = ((ParameterizedType) genType).getActualTypeArguments();
        if (index >= params.length || index < 0) {
            throw new RuntimeException("你输入的索引" + (index < 0 ? "不能小于0" : "超出了参数的总数"));
        }
        if (!(params[index] instanceof Class)) {
            return Object.class;
        }
        return (Class<?>) params[index];
    }

    /**
     * 通过反射,获得指定类的父类的第一个泛型参数的实际类型. 如BuyerServiceBean extends DaoSupport<Buyer>
     *
     * @param clazz clazz 需要反射的类,该类必须继承泛型父类
     * @return 泛型参数的实际类型, 如果没有实现ParameterizedType接口，即不支持泛型，所以直接返回
     * <code>Object.class</code>
     */
    public static Class<?> getSuperClassGenericType(Class<?> clazz) {
        return getSuperClassGenericType(clazz, 0);
    }

    /**
     * 通过反射,获得方法返回值泛型参数的实际类型. 如: public Map<String, Buyer> getNames(){}
     *
     * @param method 方法
     * @param index  泛型参数所在索引,从0开始.
     * @return 泛型参数的实际类型, 如果没有实现ParameterizedType接口，即不支持泛型，所以直接返回
     * <code>Object.class</code>
     */
    public static Class<?> getMethodGenericReturnType(Method method, int index) {
        Type returnType = method.getGenericReturnType();
        return getClass(index, returnType);
    }

    /**
     * 通过反射,获得方法返回值第一个泛型参数的实际类型. 如: public Map<String, Buyer> getNames(){}
     *
     * @param method 方法
     * @return 泛型参数的实际类型, 如果没有实现ParameterizedType接口，即不支持泛型，所以直接返回
     * <code>Object.class</code>
     */
    public static Class<?> getMethodGenericReturnType(Method method) {
        return getMethodGenericReturnType(method, 0);
    }

    /**
     * 通过反射,获得方法输入参数第index个输入参数的所有泛型参数的实际类型. 如: public void add(Map<String,
     * Buyer> maps, List<String> names){}
     *
     * @param method 方法
     * @param index  第几个输入参数
     * @return 输入参数的泛型参数的实际类型集合, 如果没有实现ParameterizedType接口，即不支持泛型，所以直接返回空集合
     */
    public static List<Class<?>> getMethodGenericParameterTypes(Method method,
                                                                int index) {
        List<Class<?>> results = new ArrayList<>();
        Type[] genericParameterTypes = method.getGenericParameterTypes();
        if (index >= genericParameterTypes.length || index < 0) {
            throw new RuntimeException("你输入的索引"
                    + (index < 0 ? "不能小于0" : "超出了参数的总数"));
        }
        Type genericParameterType = genericParameterTypes[index];
        if (genericParameterType instanceof ParameterizedType) {
            ParameterizedType aType = (ParameterizedType) genericParameterType;
            Type[] parameterArgTypes = aType.getActualTypeArguments();
            for (Type parameterArgType : parameterArgTypes) {
                Class<?> parameterArgClass = (Class<?>) parameterArgType;
                results.add(parameterArgClass);
            }
            return results;
        }
        return results;
    }

    /**
     * 通过反射,获得方法输入参数第一个输入参数的所有泛型参数的实际类型. 如: public void add(Map<String, Buyer>
     * maps, List<String> names){}
     *
     * @param method 方法
     * @return 输入参数的泛型参数的实际类型集合, 如果没有实现ParameterizedType接口，即不支持泛型，所以直接返回空集合
     */
    public static List<Class<?>> getMethodGenericParameterTypes(Method method) {
        return getMethodGenericParameterTypes(method, 0);
    }

    /**
     * 通过反射,获得Field泛型参数的实际类型. 如: public Map<String, Buyer> names;
     *
     * @param field 字段
     * @param index 泛型参数所在索引,从0开始.
     * @return 泛型参数的实际类型, 如果没有实现ParameterizedType接口，即不支持泛型，所以直接返回
     * <code>Object.class</code>
     */
    public static Class<?> getFieldGenericType(Field field, int index) {
        Type genericFieldType = field.getGenericType();

        return getClass(index, genericFieldType);
    }

    /**
     * 获取参数类型
     *
     */
    private static Class<?> getClass(int index, Type genericFieldType) {
        if (genericFieldType instanceof ParameterizedType) {
            ParameterizedType aType = (ParameterizedType) genericFieldType;
            Type[] fieldArgTypes = aType.getActualTypeArguments();
            if (index >= fieldArgTypes.length || index < 0) {
                throw new RuntimeException("你输入的索引"
                        + (index < 0 ? "不能小于0" : "超出了参数的总数"));
            }
            return (Class<?>) fieldArgTypes[index];
        }
        return Object.class;
    }

    /**
     * 通过反射,获得Field泛型参数的实际类型. 如: public Map<String, Buyer> names;
     *
     * @param field 字段
     * @return 泛型参数的实际类型, 如果没有实现ParameterizedType接口，即不支持泛型，所以直接返回
     * <code>Object.class</code>
     */
    public static Class<?> getFieldGenericType(Field field) {
        return getFieldGenericType(field, 0);
    }

}

```

## 创建：`BuilderDifferenceInfoHandle.java`核心类
```java
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.util.TypeUtils;
import com.google.common.base.MoreObjects;
import com.neo.util.ArrayUtils;
import com.neo.util.ReflectUtils;
import org.apache.commons.lang3.StringUtils;
import org.reflections.Reflections;

import java.lang.annotation.Annotation;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Objects;
import java.util.Set;

/**
 * 组装修改信息工具类
 * <p>
 * 处理器优先级：BuilderDifferenceInfoHandle.Builder#putFieldNameProcessHandleMapping() > Property#typeProcessHandleUsing() > BuilderDifferenceInfoHandle.Builder#registerProcessHandle() > 内置处理器 > 默认
 * 属性名称优先级：BuilderDifferenceInfoHandle.Builder#putFieldTitleMapping() > Property#nameAnnotationClass() > Property#name() > BuilderDifferenceInfoHandle.Builder#configGlobalNameAnnotation()
 * <p>
 * 需依赖：
 * <pre>
 * <dependency>
 *      <groupId>com.google.guava</groupId>
 *      <artifactId>guava</artifactId>
 *      <version>28.0-jre</version>
 * </dependency>
 *
 * <dependency>
 *      <groupId>com.alibaba</groupId>
 *      <artifactId>fastjson</artifactId>
 *      <version>1.2.62</version>
 * </dependency>
 *
 * <dependency>
 *     <groupId>org.reflections</groupId>
 *     <artifactId>reflections</artifactId>
 *     <version>0.9.11</version>
 * </dependency>
 * </pre>
 *
 * @Author: Neo
 * @Date: 2019/11/22 22:49
 * @Version: 1.0
 */
public class BuilderDifferenceInfoHandle {

    private BuilderDifferenceInfoHandle() {
    }

    public static final String[] EMPTY_ARRAY = new String[0];
    public static TypeProcessHandle DEFAULT_TYPE_PROCESS_HANDLE = new DefaultTypeProcessHandle();

    /**
     * 内置处理器映射 ： 类型 - 处理器
     */
    public static Map<Class<?>, TypeProcessHandle> INNER_TYPE_PROCESS_HANDLE_MAP = new HashMap<>();


    /**
     * 修改前的对象
     */
    private Object oldObject;
    /**
     * 修改后的对象
     */
    private Object newObject;
    /**
     * 需要比较的字段
     */
    private String[] compareFields;
    /**
     * 比较时需要忽略的字段
     */
    private String[] ignoreCompareFields;
    /**
     * 映射 ： 类型 - 处理器
     */
    private Map<Class<?>, TypeProcessHandle> typeProcessHandleMap;
    /**
     * 映射 ： 属性名 - 处理器
     */
    private Map<String, TypeProcessHandle> fieldNameProcessHandleMap;

    /**
     * 存储修改前值的 Key
     */
    private String oldValueKey;
    /**
     * 存储修改后值的 Key
     */
    private String newValueKey;
    /**
     * 存储属性名的 Key
     */
    private String fieldNameKey;
    /**
     * 存储属性标题的 KEY
     */
    private String fieldTitleKey;
    /**
     * 是否需要组装 null 值
     */
    private boolean isBuilderNullValue;

    /**
     * 映射 ： 属性名 - 属性标题
     */
    private Map<String, String> fieldTitleMapping;

    /**
     * 全局名称标注注解类，单独使用无效，需配合 globalNameAnnotationClassField 一起使用
     */
    private Class<? extends Annotation> globalNameAnnotationClass;
    /**
     * 全局名称标注注解类属性，单独使用无效，需配合 globalNameAnnotationClass 一起使用
     */
    private String globalNameAnnotationClassField;

    private BuilderDifferenceInfoHandle(Builder builder) {
        setOldObject(builder.oldObject);
        setNewObject(builder.newObject);
        setCompareFields(builder.compareFields);
        setTypeProcessHandleMap(builder.typeProcessHandleMap);
        setFieldNameProcessHandleMap(builder.fieldNameProcessHandleMap);
        setOldValueKey(builder.oldValueKey);
        setNewValueKey(builder.newValueKey);
        setFieldNameKey(builder.fieldNameKey);
        setFieldTitleKey(builder.fieldTitleKey);
        setBuilderNullValue(builder.isBuilderNullValue);
        setFieldTitleMapping(builder.fieldTitleMapping);
        setIgnoreCompareFields(builder.ignoreCompareFields);
        setGlobalNameAnnotationClass(builder.globalNameAnnotationClass);
        setGlobalNameAnnotationClassField(builder.globalNameAnnotationClassField);
    }

    static {
        // 初始化内置处理器，仅扫描当前包路径下的类
        Reflections reflections = new Reflections(BuilderDifferenceInfoHandle.class.getPackage().getName());
        Set<Class<? extends TypeProcessHandle>> handles = reflections.getSubTypesOf(TypeProcessHandle.class);

        for (Class<? extends TypeProcessHandle> handle : handles) {
            if (handle.isAnnotationPresent(InnerTypeProcessHandle.class)) {
                try {
                    TypeProcessHandle typeProcessHandle = handle.newInstance();
                    INNER_TYPE_PROCESS_HANDLE_MAP.put(typeProcessHandle.supportTypeKey(), typeProcessHandle);
                } catch (InstantiationException e) {
                    e.printStackTrace();
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
            }
        }
    }


    /**
     * 组装对象变动信息
     *
     * @Author: Neo
     * @Date: 2019/11/22 21:59
     * @Version: 1.0
     */
    public JSONArray builderDifferenceInfo(Object oldObject, Object newObject, String... compareFields) {
        if (Objects.isNull(newObject) || Objects.isNull(oldObject)) {
            throw new RuntimeException("参数异常");
        }
        JSONArray jsonArray = new JSONArray();
        try {
            for (String field : compareFields) {

                Object oldValue = ReflectUtils.getValueByFieldName(oldObject, field);
                Object newValue = ReflectUtils.getValueByFieldName(newObject, field);

                if (!isDifferent(oldObject, newObject, field, newValue, oldValue)) {
                    continue;
                }
                jsonArray.add(builderDifferenceInfo(oldObject, newObject, field, newValue, oldValue));
            }
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
        return jsonArray;
    }


    /**
     * 组装对象变动信息
     *
     * @Author: Neo
     * @Date: 2019/11/22 21:57
     * @Version: 1.0
     */
    public JSONObject builderDifferenceInfo(Object oldObject, Object newObject, String fieldName, Object oldValue, Object newValue)
            throws IllegalAccessException, InstantiationException, NoSuchFieldException {
        String title = getFieldTitle(newObject, fieldName);
        if (StringUtils.isBlank(title)) {
            title = getFieldTitle(oldObject, fieldName);
        }
        if (StringUtils.isBlank(title)) {
            title = getFieldTitle(oldObject, fieldName);
        }

        // 获取处理器
        TypeProcessHandle handle = getTypeProcessHandle(oldObject, newObject, fieldName);
        if (Objects.isNull(handle)) {
            throw new RuntimeException("没有找到处理器");
        }

        // 调用处理器格式化方式
        Object after = handle.format(oldObject.getClass(), fieldName, oldValue);
        Object before = handle.format(newObject.getClass(), fieldName, newValue);


        JSONObject json = new JSONObject();
        json.put(this.fieldTitleKey, this.isBuilderNullValue ? MoreObjects.firstNonNull(title, StringUtils.EMPTY) : title);
        json.put(this.fieldNameKey, this.isBuilderNullValue ? MoreObjects.firstNonNull(fieldName, StringUtils.EMPTY) : fieldName);
        json.put(this.oldValueKey, this.isBuilderNullValue ? MoreObjects.firstNonNull(before, StringUtils.EMPTY) : before);
        json.put(this.newValueKey, this.isBuilderNullValue ? MoreObjects.firstNonNull(after, StringUtils.EMPTY) : after);
        return json;
    }


    /**
     * 得到类型处理器
     * 处理器优先级：BuilderDifferenceInfoHandle.Builder#putFieldNameProcessHandleMapping() > Property#typeProcessHandleUsing() > BuilderDifferenceInfoHandle.Builder#registerProcessHandle() > 默认
     *
     * @Author: Neo
     * @Date: 2019/11/24 17:41
     * @Version: 1.0
     */
    private TypeProcessHandle getTypeProcessHandle(Object oldObject, Object newObject, String fieldName) throws IllegalAccessException, InstantiationException {
        // 判断比较值得类型是否一致，都不为 null 且类型不一致则直接返回默认处理器
        Class<?> oldObjectFieldClass = getObjectFieldClass(oldObject, fieldName);
        Class<?> newObjectFieldClass = getObjectFieldClass(newObject, fieldName);

        if (!Objects.isNull(oldObjectFieldClass) && !Objects.isNull(newObjectFieldClass) && !oldObjectFieldClass.equals(newObjectFieldClass)) {
            return DEFAULT_TYPE_PROCESS_HANDLE;
        }

        // 1.BuilderDifferenceInfoHandle.Builder#putFieldNameProcessHandleMapping()
        TypeProcessHandle handle;
        if (!Objects.isNull(this.fieldNameProcessHandleMap) && !this.fieldNameProcessHandleMap.isEmpty()) {
            handle = this.fieldNameProcessHandleMap.get(fieldName);
            if (!Objects.isNull(handle)) {
                return handle;
            }
        }


        // 2.Property#typeProcessHandleUsing()
        handle = getPropertyDefinitionProcessHandle(newObject, fieldName);
        if (!Objects.isNull(handle)) {
            return handle;
        }
        handle = getPropertyDefinitionProcessHandle(oldObject, fieldName);
        if (!Objects.isNull(handle)) {
            return handle;
        }
        Class<?> targetType = MoreObjects.firstNonNull(newObjectFieldClass, oldObjectFieldClass);

        // 3.BuilderDifferenceInfoHandle.Builder#registerProcessHandle()
        if (!Objects.isNull(this.typeProcessHandleMap) && !this.typeProcessHandleMap.isEmpty()) {
            handle = this.typeProcessHandleMap.get(targetType);
            if (!Objects.isNull(handle)) {
                return handle;
            }
        }

        // 4.内置处理器
        handle = INNER_TYPE_PROCESS_HANDLE_MAP.get(targetType);
        if (!Objects.isNull(handle)) {
            return handle;
        }
        // 5.默认处理器
        return DEFAULT_TYPE_PROCESS_HANDLE;
    }

    /**
     * 获取对象属性类
     *
     * @Author: Neo
     * @Date: 2019/11/25 10:27
     * @Version: 1.0
     */
    private Class<?> getObjectFieldClass(Object object, String fieldName) {
        if (Objects.isNull(object) || StringUtils.isBlank(fieldName)) {
            return null;
        }

        if (Map.class.isAssignableFrom(object.getClass())) {
            Map map = (Map) object;
            Object value = map.get(fieldName);
            if (Objects.isNull(value)) {
                return null;
            }
            return value.getClass();
        }

        Field field = ReflectUtils.getFieldByFieldName(object, fieldName);
        if (Objects.isNull(field)) {
            return null;
        }

        return field.getType();
    }

    /**
     * 获取属性注解定义的处理器
     *
     * @Author: Neo
     * @Date: 2019/11/25 10:08
     * @Version: 1.0
     */
    private TypeProcessHandle getPropertyDefinitionProcessHandle(Object object, String fieldName) throws IllegalAccessException, InstantiationException {
        if (Objects.isNull(object) || StringUtils.isBlank(fieldName) || Map.class.isAssignableFrom(object.getClass())) {
            return null;
        }

        Field field = ReflectUtils.getFieldByFieldName(object.getClass(), fieldName);
        if (Objects.isNull(field)) {
            return null;
        }
        // 获取属性注解处理类
        Property annotation = field.getAnnotation(Property.class);
        if (Objects.isNull(annotation) || Void.class.equals(annotation.typeProcessHandleUsing())) {
            return null;
        }

        if (!TypeProcessHandle.class.isAssignableFrom(annotation.typeProcessHandleUsing())) {
            throw new RuntimeException(Property.class + ".typeProcessHandleUsing:" + annotation.typeProcessHandleUsing() + "不是 TypeProcessHandle 的实现类");
        }
        return (TypeProcessHandle) annotation.typeProcessHandleUsing().newInstance();
    }

    /**
     * 得到属性标题
     * <p>
     * 属性名称优先级：BuilderDifferenceInfoHandle.Builder#putFieldTitleMapping() > Property#nameAnnotationClass() > Property#name() > BuilderDifferenceInfoHandle.Builder#configGlobalNameAnnotation()
     *
     * @Author: Neo
     * @Date: 2019/11/22 22:30
     * @Version: 1.0
     */
    public String getFieldTitle(Object object, String fieldName) throws IllegalAccessException, NoSuchFieldException {
        if (StringUtils.isBlank(fieldName)) {
            return null;
        }
        String title = StringUtils.EMPTY;
        // 1.BuilderDifferenceInfoHandle.Builder#putFieldTitleMapping()
        if (!Objects.isNull(this.fieldTitleMapping) && !this.fieldTitleMapping.isEmpty()) {
            title = this.fieldTitleMapping.get(fieldName);
        }

        if (StringUtils.isNotBlank(title)) {
            return title;
        }

        if (Objects.isNull(object) || Map.class.isAssignableFrom(object.getClass())) {
            return null;
        }

        Field field = ReflectUtils.getFieldByFieldName(object.getClass(), fieldName);
        if (Objects.isNull(field)) {
            return null;
        }


        if (field.isAnnotationPresent(Property.class)) {
            Property property = field.getAnnotation(Property.class);
            // 2.Property#nameAnnotationClass()
            if (StringUtils.isNotBlank(property.nameAnnotationClassField())) {
                // 获取 Property.nameAnnotationClassField 标注的注解属性值
                title = getNameAnnotationTitle(field, property);

                if (StringUtils.isNotBlank(title)) {
                    return title;
                }
            }

            // 3.Property#name()
            if (StringUtils.isNotBlank(property.name())) {
                return property.name();
            }
        }

        // 4.BuilderDifferenceInfoHandle.Builder#configGlobalNameAnnotation()
        if (!Objects.isNull(this.globalNameAnnotationClass)
                && StringUtils.isNotBlank(this.globalNameAnnotationClassField)
                && field.isAnnotationPresent(this.globalNameAnnotationClass)) {
            Object annotationFieldValue = getAnnotationFieldValue(field.getAnnotation(this.globalNameAnnotationClass), this.globalNameAnnotationClassField);
            if (!Objects.isNull(annotationFieldValue)) {
                return TypeUtils.castToString(annotationFieldValue);
            }
        }
        return null;
    }

    /**
     * 获取 Property.nameAnnotationClassField 标注的注解属性值
     *
     * @Author: Neo
     * @Date: 2019/11/23 16:26
     * @Version: 1.0
     */
    public String getNameAnnotationTitle(Field field, Property property) throws NoSuchFieldException, IllegalAccessException {
        if (Void.class.equals(property.nameAnnotationClass())) {
            return null;
        }
        if (!Annotation.class.isAssignableFrom(property.nameAnnotationClass())) {
            throw new RuntimeException(Property.class + ".nameAnnotationClass:" + property.nameAnnotationClass() + "不是注解类");
        }
        Class<? extends Annotation> nameAnnotationClass = (Class<? extends Annotation>) property.nameAnnotationClass();

        // 获取 Property.nameAnnotationClass 标注的注解代理实例
        Annotation nameAnnotation = field.getAnnotation(nameAnnotationClass);
        Object annotationFieldValue = getAnnotationFieldValue(nameAnnotation, property.nameAnnotationClassField());
        return TypeUtils.castToString(annotationFieldValue);
    }

    /**
     * 获取代理注解的属性值
     *
     * @Author: Neo
     * @Date: 2019/11/25 17:07
     * @Version: 1.0
     */
    public Object getAnnotationFieldValue(Annotation annotation, String field) throws IllegalAccessException, NoSuchFieldException {
        //获取代理实例所持有的 InvocationHandler
        InvocationHandler invocationHandler = Proxy.getInvocationHandler(annotation);

        // 获取 AnnotationInvocationHandler 的 memberValues 字段
        Field memberValuesField = invocationHandler.getClass().getDeclaredField("memberValues");

        // 打开访问权限
        memberValuesField.setAccessible(true);

        // 获取 memberValues
        Map memberValues = (Map) memberValuesField.get(invocationHandler);

        // 获取 Property.nameAnnotationClassField 标注的注解属性值
        return memberValues.get(field);
    }


    /**
     * 比较值
     * 相同返回：false
     * 不相同返回：true
     *
     * @Author: Neo
     * @Date: 2019/11/22 17:58
     * @Version: 1.0
     */
    public boolean isDifferent(Object oldObject, Object newObject, String field, Object oldValue, Object newValue) throws IllegalAccessException, InstantiationException {
        // 获取处理器
        TypeProcessHandle handle = getTypeProcessHandle(oldObject, newObject, field);
        if (Objects.isNull(handle)) {
            throw new RuntimeException("没有找到处理器");
        }

        return handle.isDifferent(oldValue, newValue);
    }


    public Object getNewObject() {
        return newObject;
    }

    public void setNewObject(Object newObject) {
        this.newObject = newObject;
    }

    public Object getOldObject() {
        return oldObject;
    }

    public void setOldObject(Object oldObject) {
        this.oldObject = oldObject;
    }

    public String[] getCompareFields() {
        return compareFields;
    }

    public void setCompareFields(String[] compareFields) {
        this.compareFields = compareFields;
    }

    public String[] getIgnoreCompareFields() {
        return ignoreCompareFields;
    }

    public void setIgnoreCompareFields(String[] ignoreCompareFields) {
        this.ignoreCompareFields = ignoreCompareFields;
    }

    public Map<Class<?>, TypeProcessHandle> getTypeProcessHandleMap() {
        return typeProcessHandleMap;
    }

    public void setTypeProcessHandleMap(Map<Class<?>, TypeProcessHandle> typeProcessHandleMap) {
        this.typeProcessHandleMap = typeProcessHandleMap;
    }

    public Map<String, TypeProcessHandle> getFieldNameProcessHandleMap() {
        return fieldNameProcessHandleMap;
    }

    public void setFieldNameProcessHandleMap(Map<String, TypeProcessHandle> fieldNameProcessHandleMap) {
        this.fieldNameProcessHandleMap = fieldNameProcessHandleMap;
    }

    public String getOldValueKey() {
        return oldValueKey;
    }

    public void setOldValueKey(String oldValueKey) {
        this.oldValueKey = oldValueKey;
    }

    public String getNewValueKey() {
        return newValueKey;
    }

    public void setNewValueKey(String newValueKey) {
        this.newValueKey = newValueKey;
    }

    public String getFieldNameKey() {
        return fieldNameKey;
    }

    public void setFieldNameKey(String fieldNameKey) {
        this.fieldNameKey = fieldNameKey;
    }

    public String getFieldTitleKey() {
        return fieldTitleKey;
    }

    public void setFieldTitleKey(String fieldTitleKey) {
        this.fieldTitleKey = fieldTitleKey;
    }

    public boolean isBuilderNullValue() {
        return isBuilderNullValue;
    }

    public void setBuilderNullValue(boolean builderNullValue) {
        isBuilderNullValue = builderNullValue;
    }

    public Map<String, String> getFieldTitleMapping() {
        return fieldTitleMapping;
    }

    public void setFieldTitleMapping(Map<String, String> fieldTitleMapping) {
        this.fieldTitleMapping = fieldTitleMapping;
    }

    public Class<? extends Annotation> getGlobalNameAnnotationClass() {
        return globalNameAnnotationClass;
    }

    public void setGlobalNameAnnotationClass(Class<? extends Annotation> globalNameAnnotationClass) {
        this.globalNameAnnotationClass = globalNameAnnotationClass;
    }

    public String getGlobalNameAnnotationClassField() {
        return globalNameAnnotationClassField;
    }

    public void setGlobalNameAnnotationClassField(String globalNameAnnotationClassField) {
        this.globalNameAnnotationClassField = globalNameAnnotationClassField;
    }

    public static Builder Builder(Object oldObject, Object newObject) {
        Builder builder = new Builder();
        builder.oldObject(oldObject);
        builder.newObject(newObject);
        return builder;
    }

    public static Builder Builder(Object oldObject, Object newObject, String... compareFields) {
        Builder builder = new Builder();
        builder.oldObject(oldObject);
        builder.newObject(newObject);
        builder.compareFields(compareFields);
        return builder;
    }


    public static final class Builder {
        /**
         * 修改前的对象
         */
        private Object oldObject;
        /**
         * 修改后的对象
         */
        private Object newObject;
        /**
         * 需要比较的字段
         */
        private String[] compareFields;
        /**
         * 比较时需要忽略的字段
         */
        private String[] ignoreCompareFields;
        /**
         * 映射 ： 类型 - 处理器
         */
        private Map<Class<?>, TypeProcessHandle> typeProcessHandleMap;
        /**
         * 映射 ： 属性名 - 处理器
         */
        private Map<String, TypeProcessHandle> fieldNameProcessHandleMap;

        /**
         * 存储修改前值的 Key
         */
        private String oldValueKey = "before";
        /**
         * 存储修改后值的 Key
         */
        private String newValueKey = "after";
        /**
         * 存储属性名的 Key
         */
        private String fieldNameKey = "name";
        /**
         * 存储属性标题的 KEY
         */
        private String fieldTitleKey = "title";
        /**
         * 是否需要组装 null 值
         */
        private boolean isBuilderNullValue = true;

        /**
         * 映射 ： 属性名 - 属性标题
         */
        private Map<String, String> fieldTitleMapping;

        /**
         * 全局名称标注注解类，单独使用无效，需配合 globalNameAnnotationClassField 一起使用
         */
        private Class<? extends Annotation> globalNameAnnotationClass;
        /**
         * 全局名称标注注解类属性，单独使用无效，需配合 globalNameAnnotationClass 一起使用
         */
        private String globalNameAnnotationClassField;

        public Builder() {
        }

        public Builder newObject(Object newObject) {
            this.newObject = newObject;
            return this;
        }

        public Builder oldObject(Object oldObject) {
            this.oldObject = oldObject;
            return this;
        }

        public Builder compareFields(String... compareFields) {
            this.compareFields = compareFields;
            return this;
        }

        public Builder ignoreCompareFields(String... typeProcessHandleMap) {
            this.ignoreCompareFields = typeProcessHandleMap;
            return this;
        }

        public Builder typeProcessHandleMap(Map<Class<?>, TypeProcessHandle> typeProcessHandleMap) {
            this.typeProcessHandleMap = typeProcessHandleMap;
            return this;
        }

        public Builder fieldNameProcessHandleMap(Map<String, TypeProcessHandle> fieldNameProcessHandleMap) {
            this.fieldNameProcessHandleMap = fieldNameProcessHandleMap;
            return this;
        }

        public Builder oldValueKey(String oldValueKey) {
            this.oldValueKey = oldValueKey;
            return this;
        }

        public Builder newValueKey(String newValueKey) {
            this.newValueKey = newValueKey;
            return this;
        }

        public Builder fieldNameKey(String fieldNameKey) {
            this.fieldNameKey = fieldNameKey;
            return this;
        }

        public Builder fieldTitleKey(String fieldTitleKey) {
            this.fieldTitleKey = fieldTitleKey;
            return this;
        }

        public Builder isBuilderNullValue(boolean isBuilderNullValue) {
            this.isBuilderNullValue = isBuilderNullValue;
            return this;
        }

        public Builder fieldTitleMapping(Map<String, String> fieldTitleMapping) {
            this.fieldTitleMapping = fieldTitleMapping;
            return this;
        }

        public Builder globalNameAnnotationClass(Class<? extends Annotation> globalNameAnnotationClass) {
            this.globalNameAnnotationClass = globalNameAnnotationClass;
            return this;
        }

        public Builder globalNameAnnotationClassField(String globalNameAnnotationClassField) {
            this.globalNameAnnotationClassField = globalNameAnnotationClassField;
            return this;
        }


        public Builder registerProcessHandle(TypeProcessHandle handle) {
            if (!Objects.isNull(handle)) {
                if (null == this.typeProcessHandleMap) {
                    this.typeProcessHandleMap = new HashMap<>(16);
                }
                this.typeProcessHandleMap.put(handle.supportTypeKey(), handle);
            }
            return this;
        }

        public Builder putFieldNameProcessHandleMapping(String field, TypeProcessHandle handle) {
            if (StringUtils.isNotBlank(field) && StringUtils.isNotBlank(field)) {
                if (null == this.fieldNameProcessHandleMap) {
                    this.fieldNameProcessHandleMap = new HashMap<>(16);
                }
                this.fieldNameProcessHandleMap.put(field, handle);
            }
            return this;
        }

        public Builder configGlobalNameAnnotation(Class<? extends Annotation> annotation, String field) {
            if (Objects.isNull(annotation) || StringUtils.isBlank(field)) {
                throw new RuntimeException("全局名称标注注解类和全局名称标注注解类属性都不可为空");
            }
            this.globalNameAnnotationClass = annotation;
            this.globalNameAnnotationClassField = field;
            return this;
        }

        public Builder putFieldTitleMapping(String field, String title) {
            if (StringUtils.isNotBlank(field) && StringUtils.isNotBlank(title)) {
                if (null == this.fieldTitleMapping) {
                    this.fieldTitleMapping = new HashMap<>();
                }
                this.fieldTitleMapping.put(field, title);
            }
            return this;
        }


        /**
         * 构建修改信息
         *
         * @Author: Neo
         * @Date: 2019/11/25 11:28
         * @Version: 1.0
         */
        public JSONArray buildInfo() {
            BuilderDifferenceInfoHandle handle = new BuilderDifferenceInfoHandle(this);
            String[] compareFields = mergeCompareFields(handle);
            if (ArrayUtils.isEmpty(compareFields)) {
                throw new RuntimeException("没有需要比较的字段");
            }
            return handle.builderDifferenceInfo(handle.getNewObject(), handle.getOldObject(), compareFields);
        }

        /**
         * 构建对象信息
         *
         * @Author: Neo
         * @Date: 2019/11/25 11:29
         * @Version: 1.0
         */
        public BuilderDifferenceInfoHandle buildObject() {
            return new BuilderDifferenceInfoHandle(this);
        }

        /**
         * 合并比较字段
         *
         * @Author: Neo
         * @Date: 2019/11/23 16:48
         * @Version: 1.0
         */
        private String[] mergeCompareFields(BuilderDifferenceInfoHandle handle) {
            // 已指定需要比较的字段并且没有需要忽略的字段
            if (ArrayUtils.isNotEmpty(handle.getCompareFields()) && ArrayUtils.isEmpty(handle.getIgnoreCompareFields())) {
                return handle.getCompareFields();
            }

            // 需要比较的字段和需要忽略的字段都指定
            if (ArrayUtils.isNotEmpty(handle.getCompareFields()) && ArrayUtils.isNotEmpty(handle.getIgnoreCompareFields())) {
                return mergeFieldName(handle.getCompareFields(), null, handle.getIgnoreCompareFields());
            }

            return mergeObjectFields(handle.getNewObject(), handle.getOldObject(), handle.ignoreCompareFields);
        }

        /**
         * 合并两个对象的属性排除忽略属性
         *
         * @Author: Neo
         * @Date: 2019/11/24 16:41
         * @Version: 1.0
         */
        private String[] mergeObjectFields(Object o1, Object o2, String[] ignoreFields) {
            if (Map.class.isAssignableFrom(o1.getClass()) && Map.class.isAssignableFrom(o2.getClass())) {
                return EMPTY_ARRAY;
            }

            if (!Map.class.isAssignableFrom(o1.getClass()) && Map.class.isAssignableFrom(o2.getClass())) {
                String[] compareFields = getFieldNames(ReflectUtils.getAllFields(o1.getClass()));
                return mergeFieldName(compareFields, null, ignoreFields);
            }

            if (Map.class.isAssignableFrom(o1.getClass()) && !Map.class.isAssignableFrom(o2.getClass())) {
                String[] compareFields = getFieldNames(ReflectUtils.getAllFields(o2.getClass()));
                return mergeFieldName(compareFields, null, ignoreFields);
            }

            if (o1.getClass().equals(o2.getClass())) {
                return getFieldNames(ReflectUtils.getAllFields(o1.getClass()));
            }

            String[] newObjectFields = getFieldNames(ReflectUtils.getAllFields(o1.getClass()));
            String[] oldObjectFields = getFieldNames(ReflectUtils.getAllFields(o2.getClass()));

            return mergeFieldName(newObjectFields, oldObjectFields, ignoreFields);
        }

        /**
         * 合并属性
         *
         * @Author: Neo
         * @Date: 2019/11/24 16:46
         * @Version: 1.0
         */
        private String[] mergeFieldName(String[] array1, String[] array2, String[] ignoreFields) {
            if (ArrayUtils.isEmpty(array1) && ArrayUtils.isEmpty(array2)) {
                return EMPTY_ARRAY;
            }
            Set<String> result = new HashSet<>(16);

            if (ArrayUtils.isNotEmpty(array1)) {
                for (String s : array1) {
                    result.add(s);
                }
            }
            if (ArrayUtils.isNotEmpty(array2)) {
                for (String s : array2) {
                    result.add(s);
                }
            }

            if (ArrayUtils.isEmpty(ignoreFields)) {
                return result.toArray(new String[0]);
            }

            for (String ignoreField : ignoreFields) {
                result.remove(ignoreField);
            }
            return result.toArray(new String[0]);
        }


        /**
         * 属性数组装换成属性名字符串数组
         *
         * @Author: Neo
         * @Date: 2019/11/24 16:46
         * @Version: 1.0
         */
        public String[] getFieldNames(Field[] fields) {
            if (ArrayUtils.isEmpty(fields)) {
                return EMPTY_ARRAY;
            }
            List<String> result = new ArrayList<>(fields.length);
            for (int i = 0; i < fields.length; i++) {
                Field field = fields[i];
                // 判断改属性是否忽略
                if (field.isAnnotationPresent(Property.class) && field.getAnnotation(Property.class).isIgnore()) {
                    continue;
                }
                result.add(fields[i].getName());
            }
            return result.toArray(new String[0]);
        }
    }
}
```

## 测试
### 创建测试Bean：`TestBean.java` (依赖：lombok)
```java
import com.alibaba.fastjson.annotation.JSONField;
import com.neo.util.diffinfo.Property;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.math.BigDecimal;
import java.util.Date;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class TestBean {

    @JSONField(name = "字符串")
    private String string;

    @JSONField(name = "整形")
    private Integer integer;

    @JSONField(name = "时间")
    @Property(nameAnnotationClass = JSONField.class , nameAnnotationClassField = "name")
    private Date date;

    @JSONField(name = "数字")
    @Property(isIgnore = true)
    private BigDecimal bigDecimal;

    @JSONField(name = "布尔")
    private Boolean aBoolean;
}
```

### 创建自定义类型处理器示例：`CustomerDateTypeProcessHandle.java`
```java
import com.alibaba.fastjson.util.TypeUtils;
import com.neo.util.diffinfo.TypeProcessHandle;

import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.util.GregorianCalendar;
import java.util.Objects;

/**
 * 时间类型处理器
 *
 * @Author: Neo
 * @Date: 2019/11/24 17:40
 * @Version: 1.0
 */
public class CustomerDateTypeProcessHandle implements TypeProcessHandle<Date> {

    public static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

    @Override
    public boolean isDifferent(Object o1, Object o2) {
        Date d1 = TypeUtils.castToDate(o1);
        Date d2 = TypeUtils.castToDate(o2);
        d1 = clearTimeTail(d1);
        d2 = clearTimeTail(d2);

        return Objects.isNull(d1) ? Objects.isNull(d2) : d1.compareTo(d2) != 0;
    }

    @Override
    public Class<?> supportTypeKey() {
        return Date.class;
    }


    @Override
    public Object format(Class<?> clazz, String fieldName, Object value) {
        Date date = TypeUtils.castToDate(value);
        if (Objects.isNull(date)) {
            return null;
        }
        return sdf.format(date);
    }

    /**
     * 清除毫秒
     *
     * @Author: Neo
     * @Date: 2019/11/22 21:23
     * @Version: 1.0
     */
    public static Date clearTimeTail(Date date) {
        if(Objects.isNull(date)){
            return null;
        }
        Calendar cal = new GregorianCalendar();
        cal.setTime(date);
        cal.set(Calendar.MILLISECOND, 0);
        return cal.getTime();
    }
}
```

### 测试类
```java
import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.annotation.JSONField;
import com.alibaba.fastjson.util.TypeUtils;
import com.google.common.collect.ImmutableMap;
import com.neo.util.diffinfo.BuilderDifferenceInfoHandle;
import com.neo.util.diffinfo.DateTypeProcessHandle;

import java.util.Map;

public class BuilderModifyInfoHandleTest {


    public static void main(String[] args) {
        Map<String, Object> oldObject = ImmutableMap.of("string", "string",
                "integer", 2,
                "date", TypeUtils.castToDate("2019-11-22 21:32:12"),
                "bigDecimal", TypeUtils.castToBigDecimal("20"),
                "aBoolean", false

        );
        TestBean newObject = TestBean.builder()
                .string("String")
                .integer(1)
                .date(TypeUtils.castToDate("2019-11-22 21:29:34"))
                .bigDecimal(TypeUtils.castToBigDecimal("10"))
                .aBoolean(true)
                .build();
        String[] compareFields = {"string", "integer", "date", "bigDecimal", "aBoolean"};
        String[] ignoreFields = {"string", "integer"};

        JSONArray array = BuilderDifferenceInfoHandle.Builder(oldObject, newObject)
                // .compareFields(compareFields)
                // .ignoreCompareFields(ignoreFields)
                .fieldTitleKey("标题")
                .putFieldTitleMapping("string", "这是字符串")
                .registerProcessHandle(new DateTypeProcessHandle())
                .putFieldNameProcessHandleMapping("date", new DateTypeProcessHandle())
                .configGlobalNameAnnotation(JSONField.class, "name")
                .isBuilderNullValue(true)
                .buildInfo();

        System.out.println(array.toJSONString());
    }
}

```

### 测试结果
```json
[
    {"before": "2019-11-22 21:29:34", "name": "date", "after": "2019-11-22 21:32:12", "标题": "时间"},
    {"before": "true", "name": "aBoolean", "after": "false", "标题": "布尔"},
    {"before": "String", "name": "string", "after": "string", "标题": "这是字符串"},
    {"before": "1", "name": "integer", "after": "2", "标题": "整形"}
]
```

## Github 地址
[github 完整测试地址](https://github.com/Yu-Hai/spring-boot-template/blob/master/spring-boot-common/src/test/java/BuilderDifferenceInfoHandleTest.java)
