# example/map

这个示例演示了使用clang分析c代码，找出函数的信息。

- .xConfig
    包含数据的配置文件。
    - 'ISROOT=true'
        表示不会再向上一级目录引入数据

- .env
    定义了两个输入参数:
    - XSARTT__clangAST__input__praseFile='"example.c"'

        需要分析的c文件，可包含路径。

    - XSARTT__clangAST__input__buildOption='""'

        编译需要的选项， 包括宏定义和头文件路径，格式为" -Dxxx ... -I/path/to/include ... "

- clangAST.xscript

    执行的逻辑
    脚本里有注解。
    输出结果为
```json
{"clantAST":{"function":[{"function_prototype":"int (int, int)","function_name":"foo","function_para":[{"para_name":"va","para_type":"int"},{"para_name":"vb","para_type":"int"}]}]}}
```


- clangAST.sh

    任意目录执行，进入到当前目录执行clangAST.xscript，否则需要在当前目录下执行。


这个例子显示xsartt比较复杂的运用，但理解了JQ语法和xsartt的管道，也不难。
xsartt的一个特点是可以在脚本里生成脚本并执行。如：
```bash
map(
    # 这条语句的意思是从每个函数compound部分中找出parameter部分的定义。
    "find -i <:<<EOF" + .object_compound + ":> -k <:<" + "_$.REGEX_PATTERN_CLANGAST_object|.object_class=\"ParmVarDecl\":format:json_>:> -p <:<~object_Decl~>:> -g object_class -g object_id -g object_name -g object_type |\nEOF\n",
    # 上一条语句找出的结果再和函数声明部分的信息功能输出需要提取的信息。
    "jaq  <:{function_prototype:\"\(.object_type)\",function_name:\"\(.object_name)\",function_para: (.|map({para_name:.object_name,para_type:.object_type}))}:>"
)
```

