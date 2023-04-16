- POJO（ Plain Ordinary Java Object）：专指只有 setter/getter/toString 的简单类，包括 DO/DTO/BO/VO 等
- DO（ Data Object）： 专指数据库表一一对应的 POJO 类。 此对象与数据库表结构一一对应，通过 DAO 层向上传输数据源对象
- DTO（ Data Transfer Object）：数据传输对象， Service 或 Manager 向外传输的对象 
- BO（ Business Object）：业务对象，可以由 Service 层输出的封装业务逻辑的对象
- VO（ View Object）：显示层对象，通常是 Web 向模板渲染引擎层传输的对象
- Query：数据查询对象，各层接收上层的查询请求


公司把 Entity 当作 DO 用

公司拿 DTO，VO 当接口方法的参数列表参数

有时候也拿 DTO，VO 当 Service，Controller 的返回值

有时候也会拿 Entity 作为 Controller 的返回值和参数列表参数

看起来很混乱，但公司的使用手册上实际有明确规定

**service** 层的**所有**返回类的**方法**，都**只能返回实体对象 - entity**，**不能**直接**返回 VO** 对象，因为VO对象是要返回给视图层用的

xxxWrapper 包装类主要是将对象转换为 VO 对象，返回给控制层，如果需要引用 service 方法，需要使用 SpringUtil 的方式


