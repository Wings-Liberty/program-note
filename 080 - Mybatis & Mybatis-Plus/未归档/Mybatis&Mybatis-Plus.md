#还没有复习 

![[../../020 - 附件文件夹/Pasted image 20230404222222.png|700]]

# Hello—Mybatis

1. 根据 xml 配置文件（全局配置文件）创建一个 SqlSessionFactory 对象，里面有数据源的一些运行环境信息
2. sql 映射文件（mapper.xml）；配置了每一个 sql，以及 sql 的封装规则等。 
3. 将 sql 映射文件注册在全局配置文件中
4. 写代码：
	1. 根据全局配置文件得到 SqlSessionFactory；
	2. 使用 sqlSession 工厂，获取到 sqlSession 对象使用他来执行增删改查
		一个 sqlSession 就是代表和数据库的一次会话，用完关闭
	3. 使用 sql 的唯一标志来告诉 MyBatis 执行哪个 sql。sql 都是保存在 sql 映射文件中的。



上述使用`Mybatis`的方法接近底层，实际使用中不会这样使用，但是助于以后的学习



## 写配置文件

```xml
<!--mybatis-config.xml-->
<!--在和Spring整合后，此配置可在yml全局配置文件中配置-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<environments default="development">
		<environment id="development">
            <!--事务处理器-->
			<transactionManager type="JDBC" />
            <!--数据源-->
			<dataSource type="POOLED">
				<property name="driver" value="com.mysql.jdbc.Driver" />
				<property name="url" value="jdbc:mysql://localhost:3306/mybatis" />
				<property name="username" value="root" />
				<property name="password" value="123456" />
			</dataSource>
		</environment>
	</environments>
</configuration>
```

## 在 xml 中写 sql

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.mybatis.dao.EmployeeMapper">
<!-- 
namespace:名称空间;如果不是使用接口式编程的话，这个全类名随便写；如果是接口式编程的话需指定为接口的全类名
id：唯一标识
resultType：返回值类型
#{id}：接口方法的参数链表传递过来的同名参数

接口式编程中，sql 标签的 id 需要和方法名一样
public Employee getEmpById(Integer id);
-->
	<select id="getEmpById" resultType="com.atguigu.mybatis.bean.Employee">
		select id,last_name lastName,email,gender from tbl_employee where id = #{id}
	</select>
</mapper>
```

## 将上述的 xml 映射到全局配置中

在`mybatis-config.xml`的 configuration 标签中添加以下内容

```xml
<!-- 将我们写好的sql映射文件（EmployeeMapper.xml）一定要注册到全局配置文件（mybatis-config.xml）中 -->
<mappers>
    <mapper resource="EmployeeMapper.xml" />
</mappers>
```

## 写代码（直接使用 SqlSession 操作数据库）

```java
// 获取SqlSessionFactory对象
@Bean
public SqlSessionFactory getSqlSessionFactory() throws IOException {
    String resource = "mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    return new SqlSessionFactoryBuilder().build(inputStream);
}
```



```java
@Test
public void test() throws IOException {
    // 2、获取 sqlSession 实例，能直接执行已经映射的 sql 语句
    // sql 的唯一标识（mapper.xml中的标签的id属性）：statement Unique identifier matching the statement to use.
    // 执行 sql 要用的参数：parameter A parameter object to pass to the statement.
    SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();

    SqlSession openSession = sqlSessionFactory.openSession();
    try {
        Employee employee = openSession.selectOne(
            "com.atguigu.mybatis.EmployeeMapper.selectEmp", 1);
        System.out.println(employee);
    } finally {
        openSession.close();
    }
}
```



## 接口式编程（使用 Mapper 操作数据库）

问题：上述的代码中，session 对象能使用的方法有限，参数高度抽象。不好用。下面使用 mapper 接口和 mapper.xml 实现

创建 mapper 接口，每个接口对应一个mapper.xml（映射规则在 mapper.xml 里的 namespace 中设置）

```java
public interface EmployeeMapper {
   
   public Employee getEmpById(Integer id);

}
```

```java
@Test
public void test01() throws IOException {
    // 1、获取sqlSessionFactory对象
    SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
    // 2、获取sqlSession对象
    SqlSession openSession = sqlSessionFactory.openSession();
    try {
        // 3、获取接口的实现类对象
        //会为接口自动的创建一个代理对象，代理对象去执行增删改查方法
        EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
        Employee employee = mapper.getEmpById(1);
        System.out.println(mapper.getClass());
        System.out.println(employee);
    } finally {
        openSession.close();
    }

}
```

虽然 mapper 接口没有实现类，但是`Mybatis`会为这个接口生成一个代理对象，由代理对象执行方法（以后会讲）



# 全局配置文件

`settings` 中的子标签 `setting` 可配置大量的属性，例如开启驼峰命名

`typeAliases` 中配置别名，配置别名后，可在 mapper.xml 中用别名替换全类名（不建议用，因为全类名更易懂，别名易被混淆）

`typeHandlers `类型处理器，用于将 mysql 中的数据类型转为 java 中的数据类型。例如 java 中使用 String 类型的变量接收 mysql 中的 varchar 数据。低版本的 mybatis 需要在配置文件中添加类型处理器。高版本的 mybatis 中已经内置了这些处理器。同时，也可以自定义类型处理器（继承BaseTypeHandler）这样的话，可自定义 mysql 的数据类型到指定的 java 中

`plugins` 插件。是 mybatis 中的拦截器。mybatis 有四大组件，插件的作用就是拦截这四大组件执行业务逻辑（插件的拦截机制使用的是动态代理）

`environments `环境。用于配置数据源，数据库的用户名密码，驱动，事务管理器等。下面就是该标签的子标签

`transactionManager` 事务管理器

`dataSource `数据源

`databaseIdProvider` 数据库提供商标识。不同的数据库使用的语法有所不同，例如在 mapper.xm 中标明此sql是由 mysql 执行的，另一条slq是由oracle执行的。mybatis会根据指定的数据库提供商寻找数据库服务并提供sql令其执行

`mappers `用于指定映射的 mapper.xml


这些标签也是有顺序的，如果书写顺序出错会报错


以上配置均可以在 Spring 中使用 yml 或 java 代码配置


# Mybatis-映射文件



## 增删改查标签



### 增删改标签

`insert`，`update`，`delete`三个标签的属性

![[../../020 - 附件文件夹/Pasted image 20230404222247.png|700]]

`parameterType`不推荐使用，已经过时。可直接使用对象中的属性名获取到参数对象中的属性。语法`#{fieldName}`

```xml
<!-- public void updateEmp(Employee employee);  -->
<update id="updateEmp">
   update tbl_employee 
   set last_name=#{lastName},email=#{email},gender=#{gender}
   where id=#{id}
</update>

<!-- public void deleteEmpById(Integer id); -->
<delete id="deleteEmpById">
   delete from tbl_employee where id=#{id}
</delete>
```

增删改的 xml 中不能写返回类型，**其 mapper 接口中的返回类型可直接写 integer，long，boolean 基本类型或包装类**。mybatis会自动根据接口的方法中声明的返回类型获取到返回值


### 查标签

`select` 标签

- Id：唯一标识符。

  用来引用这条语句，需要和接口的方法名一致
- parameterType：参数类型

  可以不传，MyBatis 会根据 TypeHandler 自动推断

- resultType：返回值类型

  别名或者全类名，如果返回的是集合，定义集合中元素的类型。不能和 resultMap 同时使用

![[../../020 - 附件文件夹/Pasted image 20230404222305.png|700]]


#### resultType

设置接口方法的返回值

- `Emp`。mapper.xml  中指定 `resultType` 为 `Emp` 的别名或全类名即可
- `List<Emp>`。mapper.xml中指定 `resultType` 为 `Emp` 的别名或全类名即可（指定list中的元素类型，而不是指定为List类）
- `Map<String, Object>`。mapper.xml 中指定 `resultType`为`map` 。返回结果将一条记录转为map
- `Map<Integer, Emp>`。希望将多条数据封装到 Map 中，Key 是主键，Value 是`Emp`。mapper.xml 中指定`resultType`为 `Emp` 的别名或全类名，返回结果将自动封装到 Map 的 value 中。并在接口方法上使用 `@MapKey("id")` 指定Map中的Key是哪个



#### resultMap

> 自定义返回的数据类型。如果 sql 返回的结果集中的列名和 javabean 中的属性名不一致，java 将无法封装返回结果



**使用场景**

1. 数据库中的列名和 javabean 中的属性名不一致
2. sql 使用联合查询，查询 `emp` 和 `dpt`。需要将结果集封装到 java 中的 `Emp` 类（此类中有 `emp` 的属性和一个 `Dpt` 对象）中。即将 sql 查询结果集中 `dpt` 相关数据封装到 java的 `Emp` 的成员变量 `Dpt dpt` 中
3. sql 使用联合查询，将结果集封装到 java 中的 `Dpt` 类（此类有 `dpt` 的属性和一个 `List<Emp>` 对象）中


**单表查询**

```xml
<!--自定义某个javaBean的封装规则
	type：自定义规则的Java类型
	id:唯一id方便引用
	  -->
<resultMap type="com.atguigu.mybatis.bean.Employee" id="MySimpleEmp">
    <!--指定主键列的封装规则
  id定义主键会底层有优化；
  column：指定哪一列(sql中的列名)
  property：指定对应的javaBean属性（java类中的属性名）
    -->
    <id column="id" property="id"/>
    <!-- 定义普通列封装规则 -->
    <result column="last_name" property="lastName"/>
    <!-- 其他不指定的列会自动封装：我们只要写resultMap就把全部的映射规则都写上。 -->
    <result column="email" property="email"/>
    <result column="gender" property="gender"/>
</resultMap>

<!-- resultMap:自定义结果集映射规则；  -->
<!-- public Employee getEmpById(Integer id); -->
<select id="getEmpById"  resultMap="MySimpleEmp">
    select * from tbl_employee where id=#{id}
</select>
```


**联合查询（有以下几种方式）**

场景一：将查询结果封装到一个有成员变量 `Dpt dpt` 的 `Emp` 类中

```xml
<!--
  联合查询：级联属性封装结果集
   -->
<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyDifEmp">
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <result column="gender" property="gender"/>
    <result column="did" property="dept.id"/>
    <result column="dept_name" property="dept.departmentName"/>
</resultMap>
```

```xml
<!-- 
  使用 association 定义关联的单个对象的封装规则；
  -->
<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyDifEmp2">
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <result column="gender" property="gender"/>

    <!--  association 可以指定联合的 javaBean 对象
  property="dept"：指定哪个属性是联合的对象
  javaType:指定这个属性对象的类型[不能省略]
  -->
    <association property="dept" javaType="com.atguigu.mybatis.bean.Department">
        <id column="d_id" property="id"/>
        <result column="dept_name" property="departmentName"/>
    </association>
</resultMap>
```

```xml
<!-- collection：分段查询 这个 sql 将分两次查询-->
<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyEmpByStep">
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <result column="email" property="email"/>
    <result column="gender" property="gender"/>
    
    <!-- association 定义关联对象的封装规则
    select:表明当前属性是调用select指定的方法查出的结果
    column:指定将哪一列的值传给这个方法

    流程：使用 select 指定的方法（传入column指定的这列参数的值）查出对象，并封装给property指定的属性
    -->
    <association property="dept" 
                 select="com.atguigu.mybatis.dao.DepartmentMapper.getDeptById"
                 column="d_id">
    </association>
</resultMap>

<!--  public Employee getEmpByIdStep(Integer id);-->
<select id="getEmpByIdStep" resultMap="MyEmpByStep">
    select * from tbl_employee where id=#{id}
</select>
```

分段查询中 `Emp` 类中的成员变量 `Dpt `的值由分段查询中的第二条 sql 查询出来。开启懒加载后，分段查询只会执行第一条 sql。当需要成员变量 `Dpt `中的数据时才会执行（自动）第二条 sql（即使用到它时次啊会执行 sql，提高了性能）

```xml
<!--全局配置文件-->
<!--显示的指定每个我们需要更改的配置的值，即使他是默认的。防止版本更新带来的问题  -->
<setting name="lazyLoadingEnabled" value="true"/>
<setting name="aggressiveLazyLoading" value="false"/>
```


场景二：将查询结果封装到一个有成员变量 `List<Emp> emps` 的 `Dpt`类中

```xml
<!--嵌套结果集的方式，使用 collection 标签定义关联的集合类型的属性封装规则  -->
<resultMap type="com.atguigu.mybatis.bean.Department" id="MyDept">
    <id column="did" property="id"/>
    <result column="dept_name" property="departmentName"/>
    <!-- 
   collection定义关联集合类型的属性的封装规则 
   ofType:指定集合里面元素的类型
  -->
    <collection property="emps" ofType="com.atguigu.mybatis.bean.Employee">
        <!-- 定义这个集合中元素的封装规则 -->
        <id column="eid" property="id"/>
        <result column="last_name" property="lastName"/>
        <result column="email" property="email"/>
        <result column="gender" property="gender"/>
    </collection>
</resultMap>
<!-- public Department getDeptByIdPlus(Integer id); -->
<select id="getDeptByIdPlus" resultMap="MyDept">
    SELECT d.id did,d.dept_name dept_name,
    e.id eid,e.last_name last_name,e.email email,e.gender gender
    FROM tbl_dept d
    LEFT JOIN tbl_employee e
    ON d.id=e.d_id
    WHERE d.id=#{id}
</select>
```

```xml
<!-- collection：分段查询 -->
<resultMap type="com.atguigu.mybatis.bean.Department" id="MyDeptStep">
    <id column="id" property="id"/>
    <id column="dept_name" property="departmentName"/>
    <collection property="emps" 
                select="com.atguigu.mybatis.dao.EmployeeMapperPlus.getEmpsByDeptId"
                column="{deptId=id}" fetchType="lazy"></collection>
</resultMap>
<!-- public Department getDeptByIdStep(Integer id); -->
<select id="getDeptByIdStep" resultMap="MyDeptStep">
    select id,dept_name from tbl_dept where id=#{id}
</select>

<!-- 扩展：多列的值传递过去：
   将多列的值封装map传递；
   column="{key1=column1,key2=column2}"
  fetchType="lazy"：表示使用延迟加载；开启全局延迟加载后，也可在此设置为立即加载
    - lazy：延迟
    - eager：立即
-->
```


**鉴别器（感觉作用不大，了解即可）**

```xml
<!-- <discriminator javaType=""></discriminator>
  鉴别器：mybatis 可以使用 discriminator 判断某列的值，然后根据某列的值改变封装行为
  封装 Employee：
   如果查出的是女生：就把部门信息查询出来，否则不查询；
   如果是男生，把 last_name 这一列的值赋值给 email;
  -->
<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyEmpDis">
    <id column="id" property="id"/>
    <result column="last_name" property="lastName"/>
    <result column="email" property="email"/>
    <result column="gender" property="gender"/>
    <!--
    column：指定判定的列名
    javaType：列值对应的java类型  -->
    <discriminator javaType="string" column="gender">
        <!--女生  resultType:指定封装的结果类型；不能缺少。/resultMap-->
        <case value="0" resultType="com.atguigu.mybatis.bean.Employee">
            <association property="dept" 
                         select="com.atguigu.mybatis.dao.DepartmentMapper.getDeptById"
                         column="d_id">
            </association>
        </case>
        <!--男生 ;如果是男生，把last_name这一列的值赋值给email; -->
        <case value="1" resultType="com.atguigu.mybatis.bean.Employee">
            <id column="id" property="id"/>
            <result column="last_name" property="lastName"/>
            <result column="last_name" property="email"/>
            <result column="gender" property="gender"/>
        </case>
    </discriminator>
</resultMap>
```

因为可以在 java 代码中对返回的结果集中数据进行修改&过滤，所以感觉鉴别器的作用不大

`resultMap` 标签中还可以设置 `TyprHandler`


## 获取自增主键的值

```xml
<insert id="addEmp" parameterType="com.atguigu.mybatis.bean.Employee"
        useGeneratedKeys="true" keyProperty="id" databaseId="mysql">
    insert into tbl_employee(last_name,email,gender) 
    values(#{lastName},#{email},#{gender})
</insert>
```

指定 `useGeneratedKeys` 的值为 `true`

指定返回的主键的存放位置 `keyProperty`。上述示例中返回的主键存放在了 `id` 中（此 sql 对应的接口方法中传递了一个 `Employee` 对象，这个 `id` 指的就是参数对象的 id 变量）


对于不支持自增主键的 oracle，应该这样写 sql。下面的示例有点多，但基本都是注释，真正的 sql 很少

```xml
<insert id="addEmp" databaseId="oracle">
    <!-- 
  keyProperty:查出的主键值封装给javaBean的哪个属性
  order="BEFORE":当前sql在插入sql之前运行
      AFTER：当前sql在插入sql之后运行
  resultType:查出的数据的返回值类型

  BEFORE运行顺序：
   先运行selectKey查询id的sql；查出id值封装给javaBean的id属性
   在运行插入的sql；就可以取出id属性对应的值
  AFTER运行顺序：
   先运行插入的sql（从序列中取出新值作为id）；
   再运行selectKey查询id的sql；
   -->
    <selectKey keyProperty="id" order="BEFORE" resultType="Integer">
        <!-- 编写查询主键的sql语句 -->
        <!-- BEFORE-->
        select EMPLOYEES_SEQ.nextval from dual 
        <!-- AFTER：
    select EMPLOYEES_SEQ.currval from dual -->
    </selectKey>

    <!-- 插入时的主键是从序列中拿到的 -->
    <!-- BEFORE:-->
    insert into employees(EMPLOYEE_ID,LAST_NAME,EMAIL) 
    values(#{id},#{lastName},#{email<!-- ,jdbcType=NULL -->}) 
    <!-- AFTER：
  insert into employees(EMPLOYEE_ID,LAST_NAME,EMAIL) 
  values(employees_seq.nextval,#{lastName},#{email}) -->
</insert>
```


## 参数绑定

之前的参数绑定都是使用 `#{fieldName}` 获取到接口方法传递的对象的属性值

```
单个参数：mybatis不会做特殊处理，
	#{参数名/任意名}：取出参数值。
	
多个参数：mybatis 会做特殊处理。
	多个参数会被封装成 一个map，
		key：param1...paramN,或者参数的索引也可以（key不是接口方法中的变量名）
		value：传入的参数值
	#{}就是从map中获取指定的key的值；
	
	
	操作：
		方法：public Employee getEmpByIdAndLastName(Integer id,String lastName);
		错误的取值方法：#{id},#{lastName}
		正确的取值方法：#{1}, #{2} 或 #{param1}, #{param2}
	异常：
        org.apache.ibatis.binding.BindingException: 
        Parameter 'id' not found. 
        Available parameters are [1, 0, param1, param2]

【命名参数】：明确指定封装参数时map的key；@Param("id")
    方法：public Employee getEmpByIdAndLastName(@Param("id") Integer id, @Param("lastName") String lastName);
	多个参数会被封装成 一个map，
		key：使用@Param注解指定的值
		value：参数值
	#{指定的key}取出对应的参数值


POJO：
如果多个参数正好是我们业务逻辑的数据模型，我们就可以直接传入pojo；
	#{属性名}：取出传入的pojo的属性值	

Map：
如果多个参数不是业务模型中的数据，没有对应的pojo，不经常使用，为了方便，我们也可以传入 map
	#{key}：取出 map 中对应的值

TO：
如果多个参数不是业务模型中的数据，但是经常要使用，推荐来编写一个 TO（Transfer Object）数据传输对象
Page{
	int index;
	int size;
}

========================思考================================	
public Employee getEmp(@Param("id") Integer id, String lastName);
	取值：id==>#{id/param1}   lastName==>#{param2}

public Employee getEmp(Integer id, @Param("e") Employee emp);
	取值：id==>#{param1}    lastName===>#{param2.lastName/e.lastName}

## 特别注意：如果是 Collection（List、Set）类型或者是数组，
		 也会特殊处理。也是把传入的 list 或者数组封装在map中。
			key：Collection（collection）,如果是 List 还可以使用这个 key(list)
				数组(array)
public Employee getEmpById(List<Integer> ids);
	取值：取出第一个id的值：   #{list[0]}
	
========================结合源码，mybatis怎么处理参数==========================
总结：参数多时会封装 map，为了不混乱，我们可以使用 @Param 来指定封装时使用的key；
#{key} 就可以取出 map 中的值；

(@Param("id") Integer id, @Param("lastName") String lastName);
ParamNameResolver解析参数封装map的；
// 1、names：{0=id, 1=lastName}；构造器的时候就确定好了

	确定流程：
	1.获取每个标了param注解的参数的@Param的值：id，lastName；  赋值给name;
	2.每次解析一个参数给map中保存信息：（key：参数索引，value：name的值）
		name的值：
			标注了param注解：注解的值
			没有标注：
				1.全局配置：useActualParamName（jdk1.8）：name=参数名
				2.name=map.size()；相当于当前元素的索引
	{0=id, 1=lastName,2=2}
				

args【1，"Tom",'hello'】:

public Object getNamedParams(Object[] args) {
    final int paramCount = names.size();
    //1、参数为null直接返回
    if (args == null || paramCount == 0) {
      return null;
     
    //2、如果只有一个元素，并且没有Param注解；args[0]：单个参数直接返回
    } else if (!hasParamAnnotation && paramCount == 1) {
      return args[names.firstKey()];
      
    //3、多个元素或者有Param标注
    } else {
      final Map<String, Object> param = new ParamMap<Object>();
      int i = 0;
      
      //4、遍历names集合；{0=id, 1=lastName,2=2}
      for (Map.Entry<Integer, String> entry : names.entrySet()) {
      
      	//names集合的value作为key;  names集合的key又作为取值的参考args[0]:args【1，"Tom"】:
      	//eg:{id=args[0]:1,lastName=args[1]:Tom,2=args[2]}
        param.put(entry.getValue(), args[entry.getKey()]);
        
        
        // add generic param names (param1, param2, ...)param
        //额外的将每一个参数也保存到map中，使用新的key：param1...paramN
        //效果：有Param注解可以#{指定的key}，或者#{param1}
        final String genericParamName = GENERIC_NAME_PREFIX + String.valueOf(i + 1);
        // ensure not to overwrite parameter named with @Param
        if (!names.containsValue(genericParamName)) {
          param.put(genericParamName, args[entry.getKey()]);
        }
        i++;
      }
      return param;
    }
  }
}
```


## ${} 和 #{} 的区别

- #{}：可以获取 map 中的值或者 pojo 对象属性的值
- ${}：可以获取 map 中的值或者 pojo 对象属性的值；


```
select * from tbl_employee where id=${id} and last_name=#{lastName}

Preparing: select * from tbl_employee where id=2 and last_name=?
```


**区别**

- #{}：是以预编译的形式，将参数设置到 sql 语句中；PreparedStatement；**防止 sql 注入**
- ${}：取出的值直接拼装在 sql语句中；**会有安全问题**；大多情况下，我们去参数的值都应该去使用 #{}


	原生 jdbc 不支持占位符的地方我们就可以使用 ${} 进行取值
	比如分表、排序...；按照年份分表拆分
	
	select * from ${year}_salary where xxx;
	select * from tbl_employee order by ${f_name} ${order}


**#{}：更丰富的用法**

规定参数的一些规则：

- javaType、 jdbcType、 mode（存储过程，用到的时候再说）、 numericScale、resultMap、 typeHandler、 jdbcTypeName

比如在取值时：`#{id, javaType=int, jdbcType=NUMBERIC}`


jdbc 和 sql 中的数据类型的见的对应关系可见 JDBCTpye.class

jdbcType 通常需要在某种特定的条件下被设置：

在我们数据为 null 的时候，有些数据库可能不能识别 mybatis 对 null 的默认处理。比如 Oracle（报错）

JdbcType OTHER：无效的类型；因为 mybatis 对所有的 null 都映射的是原生 Jdbc 的 OTHER 类型，oracle 不能正确处理;

由于全局配置中：jdbcTypeForNull=OTHER；oracle 不支持；两种办法
1、#{email,jdbcType=OTHER};
2、jdbcTypeForNull=NULL

<setting name="jdbcTypeForNull" value="NULL"/>



## 动态 sql

- if：判断

- choose（when, otherwise）：分支选择；带了 break 的 swtich-case

  如果带了 id 就用 id 查，如果带了 lastName 就用 lastName 查，只会进入其中一个

- trim 字符串截取（where - 封装查询条件, set - 封装修改条件）

- foreach 遍历集合



### ~~where~~

```xml
<select id="getEmpsByConditionIf" resultType="com.atguigu.mybatis.bean.Employee">
    select * from tbl_employee
    <!-- where 有缺点，建议使用下面的 trim 标签-->
    <where>
        <!-- 遇见特殊符号应该去写转义字符：&&：-->
        <if test="id!=null">
            id=#{id}
        </if>
        <if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
            and last_name like #{lastName}
        </if>
        <if test="email!=null and email.trim()!=&quot;&quot;">
            and email=#{email}
        </if>
        <!-- ognl会进行字符串与数字的转换判断  "0"==0 -->
        <if test="gender==0 or gender==1">
            and gender=#{gender}
        </if>
    </where>
</select>
```



### trim

```xml
<!--public List<Employee> getEmpsByConditionTrim(Employee employee);  -->
<select id="getEmpsByConditionTrim" resultType="com.atguigu.mybatis.bean.Employee">
    select * from tbl_employee
    <!-- 后面多出的and或者or where标签不能解决 
   prefix="":前缀：trim标签体中是整个字符串拼串 后的结果。
     prefix给拼串后的整个字符串加一个前缀 
   prefixOverrides="":
     前缀覆盖： 去掉整个字符串前面多余的字符
   suffix="":后缀
     suffix给拼串后的整个字符串加一个后缀 
   suffixOverrides=""
     后缀覆盖：去掉整个字符串后面多余的字符

   -->
    <!-- 自定义字符串的截取规则 -->
    <trim prefix="where" suffixOverrides="and">
        <if test="id!=null">
            id=#{id} and
        </if>
        <if test="lastName!=null &amp;&amp; lastName!=&quot;&quot;">
            last_name like #{lastName} and
        </if>
        <if test="email!=null and email.trim()!=&quot;&quot;">
            email=#{email} and
        </if> 
        <!-- ognl会进行字符串与数字的转换判断  "0"==0 -->
        <if test="gender==0 or gender==1">
            gender=#{gender}
        </if>
    </trim>
</select>
```

### where-choose-otherwise

```xml
<!-- public List<Employee> getEmpsByConditionChoose(Employee employee); -->
<select id="getEmpsByConditionChoose" resultType="com.atguigu.mybatis.bean.Employee">
    select * from tbl_employee
    <!-- when-choose-otherwise相当于java中的if-else if-esle -->
    <where>
        <!-- 如果带了 id 就用 id 查，如果带了 lastName 就用 lastName 查;只会进入其中一个 -->
        <choose>
            <when test="id!=null">
                id=#{id}
            </when>
            <when test="lastName!=null">
                last_name like #{lastName}
            </when>
            <when test="email!=null">
                email = #{email}
            </when>
            <otherwise>
                gender = 0
            </otherwise>
        </choose>
    </where>
</select>
```

### ~~set~~

`````xml
<!--public void updateEmp(Employee employee);  -->
<update id="updateEmp">
    <!-- Set 标签的使用 有问题，建议用 trim 标签-->
    update tbl_employee 
    <set>
        <if test="lastName!=null">
            last_name=#{lastName},
        </if>
        <if test="email!=null">
            email=#{email},
        </if>
        <if test="gender!=null">
            gender=#{gender}
        </if>
    </set>
    where id=#{id} 
    <!-- 		
  Trim：更新拼串
  update tbl_employee 
  <trim prefix="set" suffixOverrides=",">
   <if test="lastName!=null">
    last_name=#{lastName},
   </if>
   <if test="email!=null">
    email=#{email},
   </if>
   <if test="gender!=null">
    gender=#{gender}
   </if>
  </trim>
  where id=#{id}  -->
</update>
`````

### foreach

```xml
<!-- public List<Employee> getEmpsByConditionForeach(List<Integer> ids); -->
<select id="getEmpsByConditionForeach" resultType="com.atguigu.mybatis.bean.Employee">
    select * from tbl_employee
    <!--
    collection：指定要遍历的集合：
     list类型的参数会特殊处理封装在map中，map的key就叫list
    item：将当前遍历出的元素赋值给指定的变量
    separator:每个元素之间的分隔符
    open：遍历出所有结果拼接一个开始的字符
    close:遍历出所有结果拼接一个结束的字符
    index:索引。遍历list的时候是index就是索引，item就是当前值
            遍历map的时候index表示的就是map的key，item就是map的值

    #{变量名}就能取出变量的值也就是当前遍历出的元素
     -->
    <!--接口方法的List<Integer>参数使用了@Param("ids")修饰-->
    <foreach collection="ids" item="item_id" separator=","
             open="where id in(" close=")">
        #{item_id}
    </foreach>
</select>

<!-- 批量保存 -->
<!--public void addEmps(@Param("emps")List<Employee> emps);  -->
<!--MySQL下批量保存：可以foreach遍历   mysql支持values(),(),()语法-->
<insert id="addEmps">
    insert into tbl_employee(
    <include refid="insertColumn"></include>
    ) 
    values
    <foreach collection="emps" item="emp" separator=",">
        (#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
    </foreach>
</insert>
```


### 内置参数

两个内置参数：

mybatis 默认有两个内置参数：`_parameter`

代表整个参数

- 单个参数：_parameter 就是这个参数
- 多个参数：参数会被封装为一个 map；`_parameter` 就是代表这个 map。可以使用 `#{_parameter.fieldName}` 取值

可以用来判断是否传入了参数，如果值为 null，可以不执行对每个参数的控制判断


`_databaseId`
如果配置了 databaseIdProvider 标签。_databaseId 就是代表当前数据库的别名 oracle/mysql

结合 `if` 标签可实现一个增删改查标签中分别写适应 `mysql` 和 `oracle` 的 sql 语句


### 抽取可复用的 sql

```xml
<!-- 
    抽取可重用的sql片段。方便后面引用 
    1、sql抽取：经常将要查询的列名，或者插入用的列名抽取出来方便引用
    2、include来引用已经抽取的sql：
    3、include还可以自定义一些property，sql标签内部就能使用自定义的属性
      include-property：取值的正确方式${prop},
      #{不能使用这种方式}
   -->
<sql id="insertColumn">
    <if test="_databaseId=='oracle'">
        employee_id,last_name,email
    </if>
    <if test="_databaseId=='mysql'">
        last_name,email,gender,d_id
    </if>
</sql>

<insert id="addEmps">
    insert into tbl_employee(
    <include refid="insertColumn"></include>
    ) 
    values
    <foreach collection="emps" item="emp" separator=",">
        (#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
    </foreach>
</insert>
```


# 缓存机制


## 概述

MyBatis 系统中默认定义了两级缓存。 

`一级缓存`（本地缓存），`二级缓存`（全局缓存）

- 默认情况下，只有一级缓存（SqlSession 级别的缓存，也称为本地缓存）开启
- 二级缓存需要手动开启和配置，他是基于 namespace 级别（每个 Mapper 接口的）的缓存
- 为了提高扩展性。MyBatis 定义了缓存接口 Cache。我们可以通过实现 Cache 接口来自定义二级缓存。比如把 二级缓存放到 Redis


## 一级缓存

`一级缓存`（本地缓存）：sqlSession 级别的缓存。一级缓存是一直开启的（无法关闭）；SqlSession 级别的一个 Map

与数据库同一次会话期间查询到的数据会放在本地缓存中。以后如果需要获取相同的数据，直接从缓存中拿，没必要再去查询数据库

举个例子

```java
@Test
public void testSecondLevelCache02() throws IOException{
    SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
    SqlSession openSession = sqlSessionFactory.openSession();
    try{
        DepartmentMapper mapper = openSession.getMapper(DepartmentMapper.class);

        Department deptById = mapper.getDeptById(1);
        System.out.println(deptById);

        Department deptById2 = mapper.getDeptById(1);
        System.out.println(deptById2);
        // 第二次查询是从二级缓存中拿到的数据，并没有发送新的sql
        System.out.println(deptById==deptById2);
    }finally{
        openSession.close();
    }
}
```

将日志设置为`debug`级别，观察日志中输出的 sql 语句

输出结果

- sql 语句只打印了一次，第二次调用 mapper 的方法时没有打印 sql（即：使用同一个`SqlSession`，mapper 同一个方法调用两次，却只执行了一次 sql）
- 上述示例中，第 14 行代码输出为`true`。即：`Department`的两个对象是相等的，mybatis 将缓存中的数据直接返回，`Department`的两个引用指向的是同一片内存

每一个`SqlSession`对象都有自己的缓存，使用不同的`SqlSession`对象，使用的缓存时不同的。想了解`SqlSession`的缓存，可进入`SqlSession`的`clearCache()`方法再找，最后会找到缓存会存在一个 map 对象中

执行查，将数据缓存（如果执行相同的 sql，mybatis 会先查缓存，没有命中缓存再查库）

执行增删改，将新数据缓存


## 二级缓存

`二级缓存`（全局缓存）：基于 namespace 级别的缓存：一个 namespace / mapper 对应一个二级缓存：

工作机制：

1. 一个会话，查询一条数据，这个数据就会被放在当前会话的一级缓存中

2. 如果会话关闭；一级缓存中的数据会被保存到二级缓存中；新的会话查询信息，就可以参照二级缓存中的内容


**配置流程**

1. 在全局配置中开启缓存

   ```xml
   <setting name="cacheEnabled" value="true"/>
   ```

2. 在需要使用缓存的 mapper.xml 中配置`<cache>`标签

   ````xml
   <cache></cache>
   ````

   ````xml
   <cache eviction="FIFO" flushInterval="60000" readOnly="false" size="1024"></cache>
   <!--  
    eviction:缓存的回收策略：
     • LRU – 最近最少使用的：移除最长时间不被使用的对象
     • FIFO – 先进先出：按对象进入缓存的顺序来移除它们
     • SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象
     • WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象
     • 默认的是 LRU。
    flushInterval：缓存刷新间隔
     缓存多长时间清空一次，默认不清空，设置一个毫秒值
    readOnly:是否只读：
     true：只读；mybatis 认为所有从缓存中获取数据的操作都是只读操作，不会修改数据
        mybatis 为了加快获取速度，直接就会将数据在缓存中的引用交给用户。不安全，速度快
     false：非只读：mybatis 觉得获取的数据可能会被修改
       mybatis 会利用序列化&反序列的技术克隆一份新的数据给你。安全，速度慢
    size：缓存存放多少元素；
    type：指定自定义缓存（实现 myabtis 的 Cache 接口的类）的全类名；
    -->
   ````

3. 将使用到的缓存的pojo实现序列化接口


**效果**

1. 查询前**先尝试在二级缓存查找**，没有命中
2. **再在一级缓存中找**，没命中
3. 查库。并将数据**放在一级缓存**中
4. **关闭会话** `SqlSession` 以及缓存中的数据**转移到二级缓存**中

![[../../020 - 附件文件夹/Pasted image 20230404222352.png|700]]

mybatis 默认用的是`PerpetualCache`


## 和缓存有关的设置 / 属性

- （全局配置文件）**cacheEnabled**=true：false：关闭缓存（二级缓存关闭）(一级缓存一直可用的)
- （mapper.xml文件）每个 select 标签都有 **useCache**="true"; false：不使用缓存（一级缓存依然使用，二级缓存不使用）
- （mapper.xml文件）每个增删改标签的：**flushCache**="true"(默认)（一级二级缓存都会清除）
  - 增删改执行完成后就会清除缓存
  - 查询标签：flushCache="false"(默认)
  - 如果 flushCache=true 每次查询之后都会清空缓存；缓存是没有被使用的
  
- `sqlSession.clearCache();`只清除当前 session 的一级缓存
- （全局配置文件）**localCacheScope**：本地缓存作用域（一级缓存）。有以下两个值可以使用
  - SESSION：当前会话的所有数据保存在会话缓存中
  - STATEMENT：可以禁用一级缓存


## 整合第三方缓存框架

mybatis 是一个持久层框架，它的主要目的是和数据库打交道，并不关注缓存

即使如此，它也有缓存机制（不过不专业）。mybatis 定义了`Cache`缓存接口，可以整合第三方缓存框架，例如 mybatis 使用 redis 作为缓存

在 github 的 mybatis 的顶级项目中，就提供有多个 XXX-Cache（mybatis 整合 XXX 作为缓存的框架，比如 Redis）项目，使用这些 mybatis 提供好的缓存框架可方便整合第三方缓存

这些缓存整合框架的内容也是比较简单的，只是一些实现了`Cache`接口的类

1. 导入第三方缓存依赖

2. 导入 mybatis 整合的第三方缓存适配依赖

3. 在 mapper.xml 的`<cache>`标签的 type 属性中指定所要使用的`Cache`的实现类

```xml
<!--指定此 mapper.xml 使用的缓存的实现类-->
<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>

<!-- 引用缓存：namespace：指定和哪个名称空间下的缓存一样 -->
<!--优点，更换缓存的实现类时只需要修改一处 mapper.xml 的配置即可，其他mapper.xml由于使用的是引用而不需要挨个修改指定的缓存实现类-->
<cache-ref namespace="com.atguigu.mybatis.dao.EmployeeMapper"/>
```

4. （可选）可能需要为缓存单独创建一个配置文件。如果真的需要的话，去官方文档上找

**流程图**

![[../../020 - 附件文件夹/Pasted image 20230404222405.png|700]]

`CachingExecutor `会在使用 `Executor` 执行查操作之前先去缓存中查找数据（先查二级缓存再查一级缓存），如果缓存没命中，使用 `Executor` 查库

其中 `CachingExecutor `可以交由整合的第三方缓存去做


# Spring 整合 Mybatis

整合步骤

1. 把 SqlSessionFactoryBean 添加到 IOC 中，并在构建 SqlSessionFactoryBean 时指定 mapper.xml 文件和 mybatis 的全局配置文件位置
2. 用`@MapperScan`注解或把 MapperScannerConfigurer 添加到 IOC 中，用于扫描 Mapper 接口


`@MapperScan`是 Spring 提供的，用于替代 MyBatis 提供的。或在每个 Mapper 接口上声明的`@Mapper`


- 如果想要每个接口都要变成实现类，那么需要在每个接口类上加上 @Mapper 注解，比较麻烦，解决这个问题用 @MapperScan
- @MapperScan 的作用：指定要变成实现类的接口所在的包，然后包下面的所有接口在编译之后都会生成相应的实现类


# Mybatis 工作原理

![[../../020 - 附件文件夹/Pasted image 20230404222422.png|700]]


# Mybatis 四大对象和插件开发

- `Executor`						执行器。用于执行增删改查操作
- `ParameterHander`					参数处理器。用于设置/set和获取/get sql中的参数
- `ResultSetHandler`				结果集处理器。由于接收sql执行的结果并将其封装为java对象
- `StatementHandler`				语句处理器。由于解析java或xml中的sql语句，将其转换为能够执行的sql语句

`TypeHandler`						类型转换器。用于将 java 中的数据类型转为sql中的类型 和 将查询结果集中的数据类型转换为java中的数据类型

![[../../020 - 附件文件夹/Pasted image 20230404222434.png|450]]

Mybatis 中的插件其实就是拦截器，能拦截四大组件的方法，在组件方法执行前进行一些增强操作


比如 PageHelper 作为分页插件，会根据传入的 page 对象为原始 sql 添加 limit 条件，并再调用一个 `count(*)` 用于统计总数据量并封装到分页查询


# Mybatis-Plus

Mybatis-Plus 在 Mybatis 原有的基础上添加了新功能，但原 Mybatis 的所有功能都还能正常使用


Mybatis-Plus 添加了如下常用功能

- 提供现成的 CURD 接口
- 提供条件对象，用条件对象构造 sql，免去了写 SQL 语句
- 提供分页插件，和插件主体。插件主体能直接拦截 Mybatis 的四大组件方法



## CRUD 接口

1. 创建实体类User.class

2. 创建 dao 层接口

   ```java
   @Mapper
   public interface UserMapper extends BaseMapper<User> {
   }
   ```

3. 创建 service 接口

   ```java
   public interface UserService extends IService<User> {
   }
   ```

4. 创建 sercive 接口的实现类

   ```java
   @Service
   public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
   }
   ```


dao 层接口和 service 层接口都直接提供了大量的 crud 方法

因为没有写 sql 文件和 xml 映射文件，所以 service 和 dao 接口自动生成的 sql 里的**表名**和**字段名**默认都以 service 和 dao 接口里指定的 dto 类里的类名作为表名，属性名作为字段名


如果 DTO 里的类名和表名不同，属性名和表的字段名不同，就需要用 mybatis-plus 注解的帮助。注解的使用方式[见这里](https://baomidou.com/pages/223848/)


## 条件对象

QueryWrapper，UpdateWrapper

以条件查询为例，QueryWrapper 提供 select_list，where_list，等值查询，逻辑查询，范围查询（in, not in, between 等），模糊查询等


## 插件

Mybatis-Plus 和 Mybatis 提供的插件相比，Mybatis-Plus 提供的插件有更少的配置，参考 [mybatis-plus](https://baomidou.com/pages/2976a3/)

```java
public interface InnerInterceptor {

    /**
     * 判断是否执行 Executor#query
     */
    default boolean willDoQuery(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
        return true;
    }

    /**
     * Executor#query 操作前置处理
     */
    default void beforeQuery(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
        // do nothing
    }

    /**
     * 判断是否执行 Executor#update
     */
    default boolean willDoUpdate(Executor executor, MappedStatement ms, Object parameter) throws SQLException {
        return true;
    }

    /**
     * Executor#update 操作前置处理
     */
    default void beforeUpdate(Executor executor, MappedStatement ms, Object parameter) throws SQLException {
        // do nothing
    }

    /**
     * StatementHandler#prepare 操作前置处理
     */
    default void beforePrepare(StatementHandler sh, Connection connection, Integer transactionTimeout) {
        // do nothing
    }

    default void setProperties(Properties properties) {
        // do nothing
    }
}
```

而 Mybatis 提供了更加细粒度的控制，但配置比较繁琐。参考 [mybatis](https://mybatis.org/mybatis-3/zh/configuration.html#%E6%8F%92%E4%BB%B6%EF%BC%88plugins%EF%BC%89)

