两种打包方式

- 用 idea 打包
- 用 spring-boot-maven 插件打包

这里只介绍第一种

[打 jar 包详尽流程](https://cloud.tencent.com/developer/article/1764737)

注意：在 Project Structure -> Artifacts 里创建的 jar 中，引入的第三方 jar 包是创建此 artifact 时的 jar 包，如果之后引入了新的 jar 包或修改了原有依赖的版本，都需要重新在 Project Structure -> Artifacts 中手动增删依赖的 jar 包或重新创建一个 artifact

