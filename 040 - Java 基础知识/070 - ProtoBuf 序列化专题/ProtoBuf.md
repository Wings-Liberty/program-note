1. 下载 protobuf 编译器，protoc
2. 编写 .proto 文件
3. 导入 protobuf-java 和 protobuf-java-util 依赖
4. 用 protoc 编译生成 Java 的 .java 文件
5. 把 .java 文件复制到项目里，用  protobuf-java 和 protobuf-java-util 依赖对其进行构建，序列化和反序列化



```shell
$ protoc --java_out=/src/main/java src/protoc/chat.proto
protoc --java_out=src/main/java src/protobuf/Student.proto
```



注意：protoc 和 protobuf-java 的版本一定要完全相同



proto 语言规则可以参考下面的博客

https://www.cnblogs.com/remixnameless/p/15665313.html

https://blog.csdn.net/vvvlan/article/details/111471420

https://blog.csdn.net/BADAO_LIUMANG_QIZHI/article/details/108667427



