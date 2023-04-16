# 需求分析

需求：类 A 的实例对象 a 需要被转为类 B 的实例对象 b

常见于 DTO 转 VO，POJO 转其他类型



# 解决方案

## 简单粗暴的手动 get 和 set

```java
B b = new B();

b.setXXX(a.getXXX());
b.setXXX(a.getXXX());
b.setXXX(a.getXXX());
// ...
```

缺点：需要手写大量简单代码，且难维护

## 类型转换工具

很多工具都提供了 BeanUtil，其中通常包含两个不同类型对象的属性复制

Spring 提供的 `BeanUtils.copyProperties(Object source, Object target);`

反射实现，只能复制同名，同类型的属性

> [!warning] 如果涉及复杂的类型转换，Spring 还提供了 BeanWrapper，但这是框架内部用的工具，显然不适合开箱即用

Htools 提供了 [类型转换工具类 - Convert](https://www.hutool.cn/docs/#/core/%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2/%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2%E5%B7%A5%E5%85%B7%E7%B1%BB-Convert?id=%e7%b1%bb%e5%9e%8b%e8%bd%ac%e6%8d%a2%e5%b7%a5%e5%85%b7%e7%b1%bb-convert) 但它更偏向于 base64 编码转换，比如全角半角转换，hex 16 进制转换，数字转汉字，字符串数组和数字数组相互转换，字符转和日期转换，数组转集合... 

如果需要复杂类型转换，需要实现特定接口。内置的类型转换器有部分用反射实现，部分用硬编码实现

# MapStruct - 复杂类型转换

工作原理：不用反射实现，编译时根据注解生成合适的 get，set 方法并在类型转换的地方调用。显然运行时比反射效率高，但会影响编译速度（影响可忽略不计）


All you have to do is to define a mapper interface which declares any required mapping methods. During compilation, MapStruct will generate an implementation of this interface. This implementation uses plain Java method invocations for mapping between source and target objects, i.e. no reflection or similar.

Compared to writing mapping code from hand, MapStruct saves time by generating code which is tedious and error-prone to write. Following a convention over configuration approach, MapStruct uses sensible defaults but steps out of your way when it comes to configuring or implementing special behavior.

1. 对于同名，同类型属性，mapstruct 直接复制
2. A 和 B 父类中的属性也被包含在复制范围内
3. 对于同名，不同类型的属性
	- mapstruct 会尝试对较为简单的类型自行转换，比如 A a 中 long 类型的 price 被转为 String 后再 set 到 B b 中 String 类型的 price中。这些代码是 mapstruct 生成的（`b.setPrice(String.valueof(a))`） ^2846a0
	- mapstruct 会尝试创建一些可选的映射方法，比如 A a 中 UserDTO 被转为 UserVO 后再被 set 到 B b 的 UserVO 同名对象里，UserDTO 转 UserVO 的代码也是 mapstruct 生成的，这都不需要用注解配置


默认情况下会调用 A 和 B 的 get，set 方法进行映射，如果不存在 get，set 方法。只要
- source 的成员是 public 且不是 static
- target 的成员是 public 且不是 static 或 final
也能实现映射时进行变量的复制

# MapStruct 和 Lombok

lombok 也是在编译期生成 get，set 代码，所以如果想让 lombok 和 MapStruct 协同工作，需要做点额外的工作

解决方案可以参考[这里](https://mapstruct.org/documentation/stable/reference/html/#_reusing_mapping_configurations:~:text=14.2.-,Lombok,-MapStruct%20works%20together)


# 实际需求


## protobuf 转 pojo

**场景**
protobuf 生成了 3 种类型：Msg.CommonMsg, Msg.InternalMsg, Msg.AckMsg

3 种类型有少量相同的属性（称为 ”共享属性“），每种类型又持有不同的属性（简称 ”独有属性“）

pojo 只有一种类型：MsgContext，定义了共享属性和所有独有属性

**需求**：期望实现 proto 和 pojo 的相互转换。两种类型持有的属性名基本都相同，但 proto 里的枚举类属性需要对应到 projo 的枚举类里

proto 对象用 Builder 模式创建，创建出来的实体类无 set 方法，想要修改实体类实例只能复原 Builder 后再修改，所以普通的 mapping 工具不能实现需求


可以参考[这里](https://github1s.com/mapstruct/mapstruct-examples/blob/main/mapstruct-protobuf3/usage/src/test/java/org/mapstruct/example/ProtobufTest.java)

# 用法


> [!NOTE] 参考文档
> 以下所有内容均参考自 [MapStruct 1.5.2.Final Reference Guide](https://mapstruct.org/documentation/stable/reference/html/)


> [!NOTE] MapStruct 有 IDE 的插件支持
> IDE 和 Eclipse 都有支持 MapStruct 编码的插件。但不是必须的，没有插件也能用 MapStruct


```xml
...
<properties>
    <org.mapstruct.version>1.5.2.Final</org.mapstruct.version>
</properties>
...
<dependencies>
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>${org.mapstruct.version}</version>
    </dependency>
</dependencies>
...
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>${org.mapstruct.version}</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
...
```



## @Mapping
- 简单的 @Mapping(source, target)
- 作用在注解上的 @Mapping，假设作用在了 @A 上，那么被 @A 注解修饰的方法直接继承 @A 上的所有 @Mapping 注解。但此功能还在试验阶段
- ignore +target 属性可以指定忽略此属性的复制


## 自定义映射方法
如果 mapstruct 生成的转换方法不能满足需求，用户可以自定义映射方法

- 在接口里自定义接口方法实现（jdk8+ 允许在接口里写有方法体的方法，但需要有 default 修饰），这常被用于复杂的复制，或**被其他 mapping 方法调用**
- @Mapper 注解也可以作用在抽象类中，抽象类天然支持定义有方法体的方法。同时抽象类还支持持有成员变量，接口就不能。@Mapping 修饰抽象方法即可让 mapstruct 生成代码


> [!NOTE] 自定义映射方法的方法名
> ~~但为了让 mapstruct 生成的 mapping f1 方法能识别并调用到自定义的 mapping f2 方法，最好把方法名定义为 `default Xxx xxToXxx(xx xx)`~~
> 
> MapStruct 获取 mapping fun 好像是根据入参类型和返回类型决定的，所以方法名可以随便自定义




> [!NOTE] 接口和抽象类的区别
> 其中一个区别是，接口只能有 public static 的共有类变量，抽象类能有成员变量


## 多参数，单一返回值的 mapping 方法

多参数的 mapping 方法目的在于：把多个参数复制，合并到一个对象里

假设有两个参数 source1, source2。把两个对象的属性汇总起来 

- 有些属性名没有重复，且需要被复制。mapstruct 会自动选择
- 有些属性需要被复制，但 source1, 2 中都出现了同名属性，这就需要在 @Mapping 上用 source 属性显式声明用哪个属性复制，比如`@Mapping(target = "description", source = "person.description")`

也有直接把参数赋值给 Result 类的方法（而不是把参数对象和结果对象的同名属性进行赋值，是把参数对象本身赋值给结果对象的同名属性）

```java
@Mapping(target = "description", source = "person.description")
@Mapping(target = "houseNumber", source = "hn")
DeliveryAddressDto personAndAddressToDeliveryAddressDto(Person person, Integer hn);
```

## 参数的属性复制到结果对象里

期望把参数 A a 的 属性 Person p 里的属性赋值给 B b 的属性里，而不仅仅是把 a.person 赋值给 b.person

设置 Mapping 时，原本的

```java
@Mapping( target = "id", source = "person.id" )
@Mapping( target = "name", source = "person.name" )
@Mapping( target = "age", source = "person.age" )
```

改为
```java
@Mapping( target = ".", source = "person" )
```

如果出现了冲突，如

```java
@Mapping( target = ".", source = "record" )
@Mapping( target = ".", source = "account" )
```

结果 a.record 和 a.account 里都有 name 属性。这需要手动解决

手动声明 b.name 的 source 来自哪

```java
@Mapping( target = "name", source = "record.name" )
@Mapping( target = ".", source = "record" )
@Mapping( target = ".", source = "account" )
```

## 指定被更新的结果对象

先前的 mapping 方法都是返回一个新对象，也可以指定一个结果对象，mapping 方法将会把 source 映射到指定的对象里

```java
// 因为两个对象的属性都是同名的，所以不用额外的 @Mapping 注释
// @MappingTarget 标注了指定的结果对象。方法的返回值可以是 void 也可以是 Car 这个结果类
void updateCarFromDto(CarDto carDto, @MappingTarget Car car);
```


## 用 Builder 构建返回不可变的对象的映射方法

[参考这里](https://mapstruct.org/documentation/stable/reference/html/#_custom_accessor_naming_strategy:~:text=project%20on%20GitHub.-,3.8.%20Using%20builders,-MapStruct%20also%20supports)


## 选择构造器

A -> B 的映射方法中，没有用 `@MappingTarget` 时，一定会创建一个新的 B 对象

先找有没有用 Builder 的构造器，如果没有，按照某些规则从所有构造器里选一个来创建对象

选择策略参考[这里](https://mapstruct.org/documentation/stable/reference/html/#_custom_accessor_naming_strategy:~:text=3.9.-,Using%20Constructors,-MapStruct%20supports%20using)


## map 转 bean

```java
@Mapping(target = "name", source = "customerName")
Customer toCustomer(Map<String, String> map);
```

在创建  map 转 Customer 的代码时也会用到 mapstruct  [[对象的类型转换 - MapStruct#^2846a0|内置的类型转换]]
- 简单类型的转换
- 尝试找一些可选的映射方法

## 创建 Mapper 的方式

用户调用类型转换时，会调用接口里的 mapper.xxToXxx()

先前定义 / 创建 mapper 时，是在接口里这样写的

```java
@Mapper
public interface CarMapper {
	CarMapper mapper = Mappers.getMapper( CarMapper.class );

	// mapping methods ...
}
```

这是最简单的方式，没有用任何依赖注入框架。仅通过 Mappers 获取

且获取到的对象是单例的，无状态的。所以也是线程安全的

也可以用 CDI 或 Spring 管理 Mapper 对象，这需要在 @Mapper#componentModel 里配置，或在启动前，在 pom 文件里进行全局配置或在命令行里进行全局配置

注入策略也可以选 field 方式或构造器注入

> 就默认的得了，别非那么大劲了。默认的已经单例，无状态了，还要啥自行车

# source 和 target 的类型转换

## 默认的隐式转换

- 原始基本类型和对应的包装类
- 原始基本类型和其他包装类
- （原始基本类型，其他包装类）和 Stirng 之间

这些之间的转换 mapstruct 将自行完成，且把包装类转为基本类型时也会进行相应的 null 判断

- 枚举类和 String
- 大数字，基本数据类型和包装类，Stirng 之间的转换
- Between `java.sql.Time` and `java.util.Date` ...（还有超多的隐式转换）

这些转换统统都能被 mapstruct 自行完成，且涉及字符串和其他（如数字，日期）之间的转换还可以定义格式化模式

## 类对象的映射

Car 持有一个 Person，像将其转为持有 PersonDTO 的 CarDTO

Car 转 CarDTO 时又会调用 Person 转 PersonDTO 的方法（最好手动定义一下）


所以 mapstruct 能完成任意深度的对象复制（也就是”深度拷贝“ - 深度拷贝的原意是对同类型对象的拷贝，这里指不同类型的复制）

在生成映射方法的实现时，MapStruct将为源和目标对象中的每个属性对应用以下例程：

- 如果 source 和 target 的类型相同，直接进行引用复制（用 @DeepClone 可进行深度映射复制）
- 如果 source 和 target 的类型不同
	- 能找到对应的 mapping 方法，就调用
	- 找不到对应的 mapping 就找隐式类型转换方案（之前说过，比如 int 转 String 时调 `String.valueof(i)`，或 String 转 int 调 `Integer.parse(str)`）
	- 还是找不到就进行复杂映射转换方案
		- 在内置转换方案和 mapping 方法里寻找方案，假设能找到**调用多次**内置转换和 mapping 方法就能实现 source 和 target 的方案就算找到
		- 还是找不到，mapstruct 就自定生成映射方法（可以用 `@Mapper( disableSubMappingMethodsGeneration = true )`将这个方案直接禁用掉），还是不行就报错


## 类型转换方法的作用域和生命周期

一个 Mapper 里能定义多个映射方法

一个映射方法里能调用其他 mapper 里的映射方法（如果 mapstruct 认为有必要的话就会调用）

所以就像是 mapstruct 为所有映射方法创建了一个对象，并把入参类型和返回类型作为 key 进行检索和使用

但为了避免用户无意覆盖掉隐式类型转换器，当一个 mapper 里需要用到其他 mapper 里的方法时，可以在 `@Mapper` 里声明

```java
@Mapper(uses=DateMapper.class)
public interface CarMapper {
    CarDto carToCarDto(Car car);
}
```

## 处理多个同类型的类型转换器

假设有两个类型转换器 c1，c2 都是 A -> B

c3 的 fun1 调用 A -> B 时发现自己 + @Mapper(uses = xxx.class) 有两个符合要求的映射方法，就会报错

为了让 fun1 能正常工作，希望给 c1, c2 的 A -> B 映射方法都起个名字，然后在 fun1 上用 @Mapping(qualifiedBy) 显式声明用哪个映射方法

为映射方法起名字有两种方式

- 用注解名表示映射方法名
- 用注解上声明的字符串值表示方法名

具体实现方式参考[这里](https://mapstruct.org/documentation/stable/reference/html/#mapping-method-resolution:~:text=5.9.-,Mapping%20method%20selection%20based%20on%20qualifiers,-In%20many%20occasions)


## 集合映射方法

用最简单的例子举例，现在需要把 Lis\t<A\> 转为 List\<B\> 或 Set\<Integer\> 转为 Set\<String\>，并且映射方法的参数类型和返回类型也是这样定义的


mapstruct 会根据泛型的入参寻找 A-> B 和隐式转换 Integer -> String 后用遍历集合并对元素执行上述找到的映射方法，得到返回结果后把结果放到新集合里

上述集合映射方法的执行过程仅适用于实现了 Iterator 接口的集合类型（Map 虽然没实现 Iterator 但工作机制类似，Map -> Map 也是遍历然后对 k，v 进行转换并加到新 map 里）


当映射方法的返回类型声明的是接口类型时，mapstruct 会创建哪个实现类作为具体的返回结果对象？这里有张对照表可以参考

![Pasted image 20220702220356](https://wings-liberty.oss-cn-beijing.aliyuncs.com/note/Pasted%20image%2020220702220356.png)


此外 mapstruct 还提供了 Stream 转集合的默认实现


> [!NOTE] 个人对 mapstruct 集合类型转换机制的看法
> 没啥用，因为不管是集合之间的转换还是集合和 Stream 之间的转换，手动用 Stream 或 foreach 实现都不困难，且复用率不高，因为集合的类型转换终究还是调用其他映射方法

## 枚举类的映射

枚举类映射到枚举类方式如下

```java
@ValueMappings({
    @ValueMapping(target = "SPECIAL", source = "EXTRA"),
    @ValueMapping(target = "DEFAULT", source = "STANDARD"),
    @ValueMapping(target = "DEFAULT", source = "NORMAL")
})
ExternalOrderType orderTypeToExternalOrderType(OrderType orderType);
```

如果两个枚举类之间的成员名非常相似，可以用 @EnumMapping 就能实现映射

比如：ChessType 和 ChessTypeSuffixed 之间的转换

```java
public enum CheeseType {
    BRIE,
    ROQUEFORT
}

public enum CheeseTypeSuffixed {
    BRIE_TYPE,
    ROQUEFORT_TYPE
}

@Mapper
public interface CheeseMapper {

    CheeseMapper INSTANCE = Mappers.getMapper( CheeseMapper.class );

    @EnumMapping(nameTransformationStrategy = "suffix", configuration = "_TYPE")
    CheeseTypeSuffixed map(CheeseType cheese);

    @InheritInverseConfiguration
    CheeseType map(CheeseTypeSuffix cheese);
}
```

@EnumMapping 的更多参数用法可参考[这里](https://mapstruct.org/documentation/stable/reference/html/#mapping-streams:~:text=8.3.-,Custom%20name%20transformation,-When%20no%20%40ValueMapping)



此外枚举类和字符串也能很简单地进行相互映射（但官方没有文档里给 demo，可能 example 项目里有）

# 对象工厂

默认情况下，在映射方法中需要创建结果对象或为结果对象创建新的成员变量时，会调用构造方法创建，但如果 @Mapper 的 uses 属性指定了 对象工厂就用工厂创建

工厂调用工厂方法创建对象，需要用 @ObjectFactory 注解修饰方法表示这是工厂方法

```java
public class DtoFactory {
     @ObjectFactory
     public CarDto createCarDto(Car car) {
         return // ... custom factory logic
     }
}

@Mapper(uses = { DtoFactory.class, EntityFactory.class, CarMapper.class } )
public interface OwnerMapper {
	// ...
}
```

# 高级选项

用于微调 mapping 方法的行为

## 为 target 配置默认值或常量值

如果 source 的某个变量值为 null 可以通过提前定义默认值，为 target 的成员赋值

假设 target 和 source 的成员名叫 mem

默认值只能是 String  类型的，如果 mem 是基本类型或其包装类型，赋值时取默认值 String 的字面值

如果 mem 是 bit，hex，bigint 等类型，只要 String 刚好符合其值的格式，也能成功赋值

```java
@Mapping(target = "stringProperty", source = "stringProp", defaultValue = "undefined")
@Mapping(target = "integerConstant", constant = "14")
```



> [!NOTE] 默认值和常量值的区别
> 默认值用于 source 的成员为 null 时使用，常量值会直接作为值赋值给 target，而不是从 source 里取值

## 表达式 - expressions

表达式仅支持 Java 语言，常用于写调用构造方法（或其他能返回对象的方法，比如静态方法或用映射方法参数里的对象调用方法）

需要注意

- 表达式仅支持 Java 语言
- 调用当前 Mapper  类上没有用 import 显式导入的类时必须写全包名。如果用 import 显式导入了类，就可以不写全包名
- 必须保证 Java 表达式的正确性，否则编译期会报错
- 在表达式里能直接访问到映射方法的参数列表里的参数对象，并能调用其方法

```java
@Mapper
public interface SourceTargetMapper {

    SourceTargetMapper INSTANCE = Mappers.getMapper( SourceTargetMapper.class );

    @Mapping(target = "timeAndFormat",
         expression = "java( new org.sample.TimeAndFormat( s.getTime(), s.getFormat() ) )")
    Target sourceToTarget(Source s);
}
```

## 默认值表达式

之前介绍过 [[对象的类型转换 - MapStruct#为 target 配置默认值或常量值|@Mapping 的 defaultValue 属性]]

在 source 的属性值为 null 时，调用默认表达式，获取默认值的返回值。默认表达式的书写方式和[[对象的类型转换 - MapStruct#表达式 - expressions|表达式]]的书写方式一样

但 defaultValue 只能设置 String 的常量，默认表达式能写代码

参考 demo 见[这里](https://mapstruct.org/documentation/stable/reference/html/#object-factories:~:text=10.3.-,Default%20Expressions,-Default%20expressions%20are)


## 子类映射 - Subclass Mapping

- 假设 Fruit 有两个子类：Apple，Banana
- 假设 FruitDTO 有两个子类：AppleDTO，BananaDTO

Fruit  只能是普通的实体类，不是接口或抽象类

```java
@Mapper
public interface FruitMapper {

    Fruit map( FruitDto source );

}
```

那么如果入参是 FruitDto，返回值就是 Fruit
如果入参是 AppleDTO，返回值还是 Fruit

因为 mapstruct 生成的代码只会调用 FruitDTO 的 get 方法和 Fruit 的 set 方法

如果希望不写新代码就实现 AppleDTO -> Apple，BananaDTO -> Banana

就这样写

```java
@Mapper
public interface FruitMapper {

    @SubclassMapping( source = AppleDto.class, target = Apple.class )
    @SubclassMapping( source = BananaDto.class, target = Banana.class )
    Fruit map( FruitDto source );

}
```

如果声明了以上 @SubclassMapping，但运行时传入了 GrapDTO，那么映射方法会因为找不到 GrapDTO 的映射方法而报错

> [!NOTE] 用了 @SubclassMapping 注解的方法返回值不能是接口或抽象类
> 如果希望返回值可以是接口或抽象类，需要设置 @Mapper#subclassExhaustiveStrategy 为 `RUNTIME_EXCEPTION`


> [!NOTE] Mapper 的配置优先级
> `@MapperConfig`，`@Mapper`，pom 配置，命令行配置


## 决定映射方法返回值类型 -  Determining the result type

- 假设 Fruit 有两个子类：Apple，Banana
- 假设 FruitDTO 有两个子类：AppleDTO，BananaDTO

```java
@Mapper( uses = FruitFactory.class )
public interface FruitMapper {

    Fruit map( FruitDto source );
    
}

public class FruitFactory {

    public Apple createApple() {
        return new Apple( "Apple" );
    }

    public Banana createBanana() {
        return new Banana( "Banana" );
    }
}

```

如果入参是 AppleDTO，返回值的实际类型会是 Fruit，Apple 还是 Banana？

默认情况下应该是 Fruit

如果希望返回值的实际类型是 Apple，需要为 map 方法加上 `@BeanMapping( resultType = Apple.class )` 注解


如果用 @Mapping 是也遇到返回值的实际类型有多种选择时，也能用 resultType 显式指定返回值的实际类型


> [!NOTE] 我的看法：这种方式应该不常用
> 因为如果指定了 resultType 说明已经明确知道需要哪种类型的返回值，那就直接用返回值类型声明就好了，而不应该在返回值类型声明处声明一个高级父类

## 控制 source 为 null 时映射方法的返回值

默认情况下 souorce 为 null 时，方法也会返回 null

如果 source 不为 null，但其成员有空值。如果在 @Mapping 上规定了 defaultValue 或 默认表达式 就为其符默认值


如果 source 为 null 时，希望让映射方法返回一个非 null 的空值，可以  `@Mapper#nullValueMappingStrategy` ，`@MapperConfig#nullValueMappingStrategy` 设置 NullValueMappingStrategy.RETURN_DEFAULT

此后，如果映射方法返回值类型是

- Map，就返回一个空 map 对象
- Iterables / Arrays，就返回一个空 Iterable 对象
- Bean，就返回一个空对象。如果有  defaultValue 或默认表达式，就执行

对于 map 和 iterables 的具体返回值类型，还有更细粒度的控制

> For collections (iterables) this can be controlled through:
> -  `MapperConfig#nullValueIterableMappingStrategy`
> -  `Mapper#nullValueIterableMappingStrategy`
> - `IterableMapping#nullValueMappingStrategy`
>   
	For maps this can be controlled through:
	- `MapperConfig#nullValueMapMappingStrategy`
	- `Mapper#nullValueMapMappingStrategy`
	- `MapMapping#nullValueMappingStrategy`

取值参考 NullValueMappingStrategy


## @TargetMapping  更新策略

如果 source.Abc 的值为 null 或 source.getAbc() 的返回类型是 void，默认情况下会给 target 的 acb 赋 null 值

更新策略能改变这一行为，更新策略在 `@Mapping`, `@BeanMapping`, `@Mapper` or `@MapperConfig` 的 nullValuePropertyMappingStrategy 上配置，而不是 @TargetMapping 上配置


- NullValuePropertyMappingStrategy.SET_TO_DEFAULT。会在 source 的成员为空时，为 target 的成员赋空值，List - ArrayList，Map - LinkedHashMap，空数组，空字符串，为基本类型赋 0 或 false 值，为 bean 创建一个对象，如果找不到合适的构造方法就用defaultValue 或默认表达式
- NullValuePropertyMappingStrategy.IGNORE。在 source 的成员为空时，不对 target 的成员进行任何修改




## 空值检查器

默认情况下，映射方法检查 source 的成员是否为 null 时，直接通过 `if(source.getXxx == null)`

如果 source 提供了 hasXxx() 方法，就会优先用 hasXxx() 方法


## 自定义检查器

如果没有任何配置，source 的成员在赋值前会进行 null 值检查

如果 source 有 hasXxx 方法，会调用这个方法代替上面的做法

如果 source 的成员有条件检查器，在复制 source 的成员到 target 的成员前，会先进行 if 检查

要求条件检查器是一个有被 @Condition 修饰的，返回 boolean 的方法。如果在接口里定义，需要加 default 才能写方法体


> [!NOTE] 在检查方法里也能传入 source

# 复用映射配置

## 继承相同配置 - @InheritConfiguration

```java
@Mapper
public interface CarMapper {

    @Mapping(target = "numberOfSeats", source = "seatCount")
    Car carDtoToCar(CarDto car);

    @InheritConfiguration
    void carDtoIntoCar(CarDto carDto, @MappingTarget Car car);
}
```

carDtoIntoCar 会继承 carDtoToCar 上的所有配置

当且仅当两个方法的 source type 和 result type 相同时注解才能生效

@InheritConfiguration 修饰的方法在寻找被继承配置的方法时，如果只有一个候选方法，@InheritConfiguration 就不需要显式指定被继承的方法；如果有多个候选方法，就需要用 @InheritConfiguration#name 指定要继承哪个方法的配置


## 反向继承 - @InheritInverseConfiguration

A -> B 和 B -> A 的配置区别通常仅在于 @Mapping 上 source 和 target 时相反的

@InheritInverseConfiguration 指定一个方法后，会生成与之相反的配置

当且仅当 A 的 source type 和 B 的 result type 相同时才会生效

```java
@Mapper
public interface CarMapper {

    @Mapping(target = "seatCount", source = "numberOfSeats")
    CarDto carToDto(Car car);

    @InheritInverseConfiguration
    Car carDtoToCar(CarDto carDto);
}
```

新的 @Mapping 配置会覆盖 @InheritInverseConfiguration 继承过来的反向配置


## 共享配置

show code

```java
@MapperConfig(
    uses = CustomMapperViaMapperConfig.class,
    unmappedTargetPolicy = ReportingPolicy.ERROR
)
public interface CentralConfig {
}
```

```java
@Mapper(config = CentralConfig.class, uses = { CustomMapperViaMapper.class } )
// 有效配置将被替换成如下配置:
// @Mapper(
//     uses = { CustomMapperViaMapper.class, CustomMapperViaMapperConfig.class },
//     unmappedTargetPolicy = ReportingPolicy.ERROR
// )
public interface SourceTargetMapper {
  ...
}
```

普通 Mapper 除了能继承 @Mapper 配置外，还能继承 @Mapping 配置

```java
@MapperConfig(
    uses = CustomMapperViaMapperConfig.class,
    unmappedTargetPolicy = ReportingPolicy.ERROR,
    mappingInheritanceStrategy = MappingInheritanceStrategy.AUTO_INHERIT_FROM_CONFIG
)
public interface CentralConfig {

    // Not intended to be generated, but to carry inheritable mapping annotations:
    @Mapping(target = "primaryKey", source = "technicalKey")
    BaseEntity anyDtoToEntity(BaseDto dto);
}
```

```java
@Mapper(config = CentralConfig.class, uses = { CustomMapperViaMapper.class } )
public interface SourceTargetMapper {

    @Mapping(target = "numberOfSeats", source = "seatCount")
    // additionally inherited from CentralConfig, because Car extends BaseEntity and CarDto extends BaseDto:
    // @Mapping(target = "primaryKey", source = "technicalKey")
    Car toCar(CarDto car)
}
```


> [!warning] 看不懂 @Mapping 的继承策略
> 看不懂 @MapperConfig#mappingInheritanceStrategy 


# 装饰器

和 aop 类似，增强映射方法的方式有两种

- [环绕增强](https://mapstruct.org/documentation/stable/reference/html/#_reusing_mapping_configurations:~:text=12.2.-,Mapping%20customization%20with%20before%2Dmapping%20and%20after%2Dmapping%20methods,-Decorators%20may%20not)
- [before 增强和 after 增强](https://mapstruct.org/documentation/stable/reference/html/#_reusing_mapping_configurations:~:text=12.2.-,Mapping%20customization%20with%20before%2Dmapping%20and%20after%2Dmapping%20methods,-Decorators%20may%20not)

## 环绕增强

需要自行实现一个装饰器类，装饰器类需要继承/实现 mapper 接口，但通常装饰器类都是抽象类。因为这样就可以选择性实现映射方法，在重写方法里增强对原方法的调用。没有被重写的方法将被 mapstruct 生成一个直接调用原方法的实现

> [!NOTE] 可能一个默认表达式就能取代装饰器
> 如果一个简单的默认表达式就能实现功能，就不需要写装饰器了

## before 和 after 增强方法

对于 before 和 after 增强方法，当方法签名的参数列表和 source 类型一致，且其他参数均符合以下要求时，方法才会被识别为增强方法

-   A parameter annotated with `@MappingTarget` is populated with the target instance of the mapping.
    
-   A parameter annotated with `@TargetType` is populated with the target type of the mapping.
    
-   Parameters annotated with `@Context` are populated with the context parameters of the mapping method.
    
-   Any other parameter is populated with a source parameter of the mapping.


当方法返回值是 void 时，增强方法会直接在调用源方法前后被调用

当方法的返回值类型和映射方法的返回值类型相同时，增强方法调用的返回结果会被映射方法的返回结果对象引用接收


> [!NOTE] 个人看法：环绕增强更强大，更容易理解，也更容易编写

如果有多个符合要求的增强方法，可以显式指定用哪些方法，指定方式见文档


# SPI 方式更换功能组件

MapStruct 中一些功能不太符合用户要求，但这些功能都是由某些组件实现的，MapStruct 用 SPI 机制允许用户自定义组件并代替默认组件

为了用 SPI 需要添加以下依赖

```xml
<dependency>  
    <groupId>org.mapstruct</groupId>  
    <artifactId>mapstruct-processor</artifactId>  
    <version>${org.mapstruct.version}</version>  
</dependency>
```

## 基于命名的变量访问器

默认情况下为 A->B 的映射方法生成的代码通过 get 和 set 访问和修改变量值

如果某些类通过 withXxx(xxx) 命名风格的方式写数据，而不是用 setXxx 的命名风格，那 mapstruct 默认情况下就会 因为找不到 get set 方法且变量不是 public 的，而在编译期或运行时抛异常

demo 参考[这里](https://github1s.com/mapstruct/mapstruct-examples/blob/HEAD/mapstruct-spi-accessor-naming/usage/src/test/java/org/mapstruct/example/SpiTest.java)


## 定义枚举类的映射方法策略

由两个 SPI 可自定义，但通常不需要用 SPI 实现这么复杂的自定义，用注解配置就已经能满足绝大多数需求了


> [!warning] 看不懂的章节
> 看不懂 5.6，5.7，5.10，6.2，10.9，13.2,（手动指定不为特定类型创建自动映射方法），13.3（自定义 BuilderProvider 去实现 Builder 的生成行为）


# 常用注释

@InheritInverseConfiguration

使用场景：为 A  -> B mapping fun1 配置了一堆映射规则，还需要创建一个 B -> A 的 mapping fun2，但映射规则仅是 fun1 的反向配置

此注解用就可用在 fun2 上，并用注解属性指定 fun1，fun2 就能以和 fun1 相反的规则生成 mapping 代码


[常用注释解释](http://www.manongjc.com/detail/18-rjstawdyltwsgze.html)

不知道 @Context 的用法，以及 @Context 在[[对象的类型转换 - MapStruct#装饰器|装饰器]]里的用法


注解配置的优先级

@Mapping > @BeanMapping > @Mapper > @MapperConfig