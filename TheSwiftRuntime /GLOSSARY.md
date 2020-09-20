# 术语表

专业术语      | 中文翻译        | 说明
:----------- | :-----------: | -----------
Entry points |        | 二进制库对外暴露的ABI接口函数
mangled |        | 为了让编译器和链接器能流畅的工作，如处理带有不同的参数类型的同名函数，SwiftObject-》_TtCs12_SwiftObject
demangled |        | mangled的逆向操作 _TtCs12_SwiftObject-》SwiftObject
more fragile |        | Swfit中修改值的任何一部分都是对于整个值的修改，意味着其中一个属性的读或写访问都需要访问整一个值。修改error stuct中一部分的修改可能会造成不可预知的错误