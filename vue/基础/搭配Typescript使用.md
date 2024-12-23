# 搭配Typescript使用

### 总览
- 官方项目脚手架create-vue，提供了Typescript就绪的选项。若自己配置tsconfig，确保编译选项中的isolateModules/strict开启，并重新配置paths。
- 基于Vite的配置中，开发服务器和打包器只会对TS做语法转义，不做类型检查。可以通过IDE配置来获得即时的类型错误反馈。可以使用vue-tsc命令为单文件组件进行类型检查和生成类型声明文件
- IDE支持：使用VSCode并添加Vue-Official扩展，该插件提供了TS支持。

### 类型声明概要
- 选项内的类型（如选项式API；没有使用\<script setup>的setup函数），需通过defineComponent函数包裹组件对象，可以推导选项内的类型。
- 使用单文件组件时，需要在\<script>标签添加lang="ts"，这样所有的模板表达式都享受类型检查；在\<script>中使用generic属性可以指定泛型参数。如：
	```
	<script setup lang="ts" generic="T,U">...</script>
	```

- 基于类型的声明：通过定义函数的泛型类型来进行类型检查。基于类型的声明会被编译器推导成运行时声明。只能用在\<script setup>中。
	```
	defineProps<{foo: string}>()
	```
- 运行时声明：通过定义函数的参数来做类型检查。运行时声明支持默认值、是否可空，同时支持JS编写的组件的类型推导。可用于所有场景，不过通常用于选项式API或非\<script setup>的场景中
	```
	defineProps(foo: {type: String, required: true})
	```
>基于类型的声明和运行时声明不可以同时使用，下面描述的都是基于类型的声明。

### 为组件Props标注类型：
```
defineProps<{foo: string}>()
```
- 可以预先创建interface，然后传入泛型参数位置。
- 默认值：在3.4以上可以使用解构语法设置；3.4及以下可以使用withDefaults(defineProps<Type>(), defaultObj)设置（其中默认值为对象时要通过getter返回）


### 为组件的emits标注类型
```
defineEmits<{
	change: [id: number, value: string]
}>()
```

### 为ref()标注类型
ref会根据初始化的值自动推断，不过也可以通过类型声明或泛型参数指定类型。如：
```
const year: Ref<string | number> = ref('2020')
const year = ref<string | number>('2020')
```
- 如果指定了类型，但没有初始值，那么实际得到的会是一个包含undefined的联合类型。

### 为reactive()标注类型
reactive会根据参数隐式推导，不过也可以通过类型声明标注（但不推荐使用泛型参数）。如：
```
const book: Book = reactive({title: 'someTitle'})
```

### 为computed()标注类型
computed会自动从计算函数的返回值上推导类型，可以通过泛型参数标注类型。如：
```
const double = computed<number>(() => {
	//若返回值不是number类型则会报错
})
```

### 为provide/inject标注类型
使用InjectionKey并传入泛型参数，来声明key，该类型继承至Symbol类型。如：
```
const key = Symbol() as InjectionKey<string>
provide(key, 'foo')
const foo = inject(key)
```
- 当调用inject时传入泛型参数，也能推断返回值的类型，通常用在key为字符串的场景。

### 为DOM引用标注类型
使用useTemplateRef时，Vue会基于匹配的元素自动推断类型。
若不能推断则可以使用泛型参数（大部分情况会自动推导）。如：
const el = useTemplateRef<HTMLInputElement>(null)

### 为自定义组件引用标注类型
使用useTemplateRef时，Vue会基于匹配的元素自动推断类型。若不能推断则可以使用泛型参数（大部分情况会自动推导）。如：
- 普通组件类型：
	```
	const compRef = useTemplateRef<InstanceType<typeof Foo>>('comp')
	```
- 不关心具体类型，只调用通用属性：
	```
	const child = useTemplateRef<ComponentPublicInstance>(null)
	```
- 泛型组件类型：
	```
	const modal = useTemplateRef<ComponentExposed<typeof MyGenericModal>>(null)
	```
	>InstanceType是TS内置专门用来获取构造函数的实例类型的工具；
	ComponentPublicInstance导入自Vue；
	ComponentExposed导入自vue-component-type-helper