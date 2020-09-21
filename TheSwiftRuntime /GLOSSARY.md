# 术语表

专业术语      | 中文翻译        | 说明
:----------- | :-----------: | -----------
Entry points |        | 二进制库对外暴露的ABI接口函数
mangled |        | 对一些符号的签名和编码，如SwiftObject编码成__TtCs12_SwiftObject
demangled |        | mangled的逆向操作，对一些符号的解码，如__TtCs12_SwiftObject解码成SwiftObject
conformance |        | 在编译过程中生成，包含struct或类对protocol方法的具体实现