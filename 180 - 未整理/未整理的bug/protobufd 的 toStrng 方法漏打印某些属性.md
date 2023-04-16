## 问题描述


场景：用 protobuf3 生成了多个 Msg.XxxMsg，直接打印对象时，有些属性不打印

```xml
<dependency>  
    <groupId>com.google.protobuf</groupId>  
    <artifactId>protobuf-java</artifactId>  
    <version>3.20.1</version>  
</dependency>
```

操作：写 proto 文件

```proto
enum Module {
  CLIENT = 0;  
  TRANSFER = 1;  
  ROUTER = 2;  
}
```

```java
Msg.InternalMsg msg = Msg.InternalMsg.newBuilder()   
        .setFrom(Msg.Module.CLIENT)  
        .setDest(Msg.Module.TRANSFER)  
        .build();  
  
System.out.println(msg);
System.out.println(msg.getFrom());
```

期望结果：

```
from: CLIENT
dest: TRANSFER
CLIENT
```

实际结果：

```
dest: TRANSFER
CLIENT
```

## 解决方案

可能导致此问题的原因：proto 生成的 Java 对象先用 int 类型变量保存枚举类，但 int 变量必须有值，不能为 null

当没有设置 From 时，这个 int 的值为 0（int 的零值）

当设置了 .setFrom(Msg.Module.CLIENT) 时，因为给 CLIENT 配的值也是 0

所以 proto 打印的时候可能误以为 0 值表示用户没有设置此值就不打印

但这只会影响对象的 toString 调用，不会影响 From 的设置，因为只要设置了 From 的值，通过 `msg.getFrom()`总是能得到值的，就如上面运行结果的最后一行

解决方案：~~**不要在 proto 文件里给枚举值设置 0 值**~~，这个方案不能解决问题