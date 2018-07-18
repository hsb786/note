## 规则
1. 应用层级的状态应该集中到单个sotre对象中
2. 提交mutation是更改状态的唯一方法，并且这个过程是同步的
3. 异步逻辑都应该封装到action里面

如果store文件太大，只需将action、mutation和getter分割到单独的文件
