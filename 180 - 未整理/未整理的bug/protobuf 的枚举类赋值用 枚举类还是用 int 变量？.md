如果用枚举类会存在一个问题

Builder 会检查 null 值，如果传入 null 就会抛异常

我在用 mapstruct 时，希望把对象 A 转为 protoA，但 map 方法在映射枚举类时，如果 A 对象的枚举类变量值是 null，map 也会把 protoA 的枚举类变量设为 null，但 Builder 在设置枚举类的 null 时会抛异常

如果调用的是 Builder 的 setEnumValue 传入 int 值就不存在这个问题了


解决方式

还未解决



