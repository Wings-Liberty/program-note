#还没有复习 

已知注意事项：

- Handler 是不共享的。所以默认无法用 Spring 复用同一个 handler  对象
- bytebuf 不能被多次访问，否则会报异常。除非调用 retain 增加其允许被访问的次数