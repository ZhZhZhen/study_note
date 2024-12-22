# 反射修改final
- 1、反射可以修改非static的final域，但是由于编译优化，使用该变量的地方会直接返回常量字段，而不是返回final的变量，所以无法查看结果。如果该final字段需要在运行期才能确定，那么不会被编译优化(比如final int a = flag? 1 : 2)。这样的final字段通过反射修改是可以查看结果的
- 2、反射修改static final域时，会抛出异常，因此需要通过反射首先修改Field中的mondifiers数据域(也是一个Field)中代表final的bit为0，才能继续反射修改