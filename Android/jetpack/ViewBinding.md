# ViewBinding
- 简述：使用APT构建代码，主要帮助省去findViewById()操作。
- 使用：通过inflate()传入LayoutInflater来返回实现类。
- inflate()：1、根据传入的LayoutInflater来解析xml生成对应布局中的顶层rootView。2、通过bind()方法去获取xml中所有View的实例（通过findViewById()）。3、最后通过生成的构造方法生成实现类返回，该构造方法的参数为所有View（所以这个构造方法参数会很长= =）
- 如果使用include：需要给include赋值id，然后通过ViewBinding.includeid.view来访问include里的View