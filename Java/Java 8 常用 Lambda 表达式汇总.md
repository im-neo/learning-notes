# Java 8 常用 Lambda 表达式汇总

## 集合操作
```java
List<Person> list = Lists.newArrayList(
                new Person("张三", "F", 20, new BigDecimal("15000")),
                new Person("李四", "M", 23, new BigDecimal("20000")),
                new Person("王五", "F", 25, new BigDecimal("25000")),
                new Person("赵六", "M", 27, new BigDecimal("30000")),
                new Person("吴七", "F", 30, new BigDecimal("35000"))
        );
        

// 1.计数
long count = list.stream().collect(Collectors.counting());

// 2.最值
Person person = list.stream().collect(Collectors.maxBy(Comparator.comparingInt(Person::getAge))).get();

// 3.求和
int sum = list.stream().collect(Collectors.summingInt(Person::getAge));

// 4.求平均值
double avg = list.stream().collect(Collectors.averagingDouble(Person::getAge));

// 5.属性分组
Map<String, List<String>> collect = list.stream()
        .collect(Collectors.groupingBy(Person::getGender, Collectors.mapping(Person::getName, Collectors.toList())));

// 6.一级分组
Map<String, List<Person>> collect1 = list.stream().collect(Collectors.groupingBy(person -> {
    if (person.getAge() >= 30) {
        return "30";
    } else if (person.getAge() >= 25) {
        return "25";
    } else {
        return "20";
    }
}));

// 7.多级分组
Map<String, Map<String, List<Person>>> collect2 = list.stream().collect(Collectors.groupingBy(p -> {
    if (p.getAge() >= 30) {
        return "30";
    } else if (p.getAge() >= 25) {
        return "25";
    } else {
        return "20";
    }
}, Collectors.groupingBy(Person::getGender)));

// 8.对分组进行统计
Map<String, Long> collect3 = list.stream().collect(Collectors.groupingBy(p -> {
    if (p.getAge() >= 30) {
        return "30";
    } else if (p.getAge() >= 25) {
        return "25";
    } else {
        return "20";
    }
}, Collectors.counting()));

// 9.将对象集合中的属性转换映射到新的集合
Set<String> names = list.stream().map(Person::getName).collect(Collectors.toSet());

// 10.将对象的一个属性作为 Map 的 key,本身作为 value 映射到 Map 集合
Map<String, Person> empMap1 = list.stream()
        .collect(Collectors.toMap(emp -> emp.getName(), Function.identity()));

// 11.将对象的一个属性作为 Map 的 key，另一个属性作为 value 映射到 Map 集合
Map<String, Integer> empMap2 = list.stream()
        .collect(Collectors.toMap(emp -> emp.getName(), Person::getAge));

// 12.通过属性过滤数据
List<Person> empList = list.stream()
        .filter(emp -> emp.getAge() >= 25 && emp.getAge() <= 30)
        .collect(Collectors.toList());

// 13.数值进行求和算数操作,并将结果四舍五入
BigDecimal totalIncome = list.stream()
        .map(Person::getIncome)
        .reduce(BigDecimal.ZERO, BigDecimal::add)
        .setScale(2, BigDecimal.ROUND_HALF_UP);

// 14.单条件排序
List<Person> sortList1 = list.stream().sorted(Comparator.comparing(Person::getAge))
        .collect(Collectors.toList());

// 15.多条件排序
List<Person> sortList2 = list.stream().sorted(Comparator.comparing(Person::getAge)
        .thenComparing(Person::getIncome).reversed())
        .collect(Collectors.toList());
```

```java
public class Person {
        private String name;
        private String gender;
        private Integer age;
        private BigDecimal income;


        public Person(String name, String gender, Integer age, BigDecimal income) {
            this.name = name;
            this.gender = gender;
            this.age = age;
            this.income = income;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getGender() {
            return gender;
        }

        public void setGender(String gender) {
            this.gender = gender;
        }

        public Integer getAge() {
            return age;
        }

        public void setAge(Integer age) {
            this.age = age;
        }

        public BigDecimal getIncome() {
            return income;
        }

        public void setIncome(BigDecimal income) {
            this.income = income;
        }
    }
```

