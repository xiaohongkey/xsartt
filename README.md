# xsartt

xsartt: xiaohong's struct & regular text tool的缩写。 意思就是用来处理文本的工具。
平时在工作中用到shell/python/java甚至是c,本质上都是符合一定规则的文本。这些工具都很强大，但用起来总要东拼西凑，所以写了这个工具。
这个工具的本意就是输入一段符合规则的文本，再用一段文本描述这段规则，输出一段意图输出的文本。
shell中的变量都是文本，但都是一维的， python等可以有结构化的数据，但比较繁琐。所以我希望有一个处理系列化后文本的工具。
这些系列化的数据可以是流行的各种格式，json/yaml/toml/xml甚至是可以用正则表达式解析的文本。这个工具主要功能就是用来处理这种输入文本。
逻辑处理我采用了JQ语法逻辑，我自己也写不出来，所以引用了一个开源的项目（https://github.com/01mf02/jaq), 包裹在这个工具里，用来处理数据。自己又写了一部分分析嵌套文本块的逻辑，共同组成这个工具的核心部分。
这个工具当初主要用处是用来代码的自动生成。也可以生成任意自己编辑模板的文本。还有一些其他应用，比如写了一个脚本语言，还写了一个make的工具，还有一些乱七八糟的功能，都是基于输入一段包含格式化的数据的文本，输出想要的文本。
我还把主要逻辑包装成了一个PYTHON库(pyxsartt),可以在PYTHON中调用。
下面就一一简单介绍。

## 子命令

### block
```bash
$ xsartt block --help
在文本中找出指定界定符的块, 可嵌套.for example:
>> echo '<~甜aa~> <~ccc <~ddd~> ~>' | xsartt block -i @-
{"range":[0,27],"str":"<~甜aa~> <~ccc <~ddd~> ~>\n","sub":[{"range":[2,7],"str":"甜aa","sub":[]}, {"range":[12,24],"str":"ccc <~ddd~> ","sub":[{"range":[18,21],"str":"ddd","sub":[]}]}]}

Usage: xsartt.exe block [OPTIONS] --input-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>

Options:
      --WTF <文件名>                                           输出到文件
  -l, --left-backet <短语>                                    文本中块的左边界限符号 [default: <~]
  -r, --right-backet <短语>                                   文本中块的右边界限符号 [default: ~>]
  -i, --input-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>  输入文本
  -h, --help 
```

### pattern
最初的目的就是是用来组合生成正则表达式。主要的逻辑是分别定义子表达式， 再组合成一个大的表达式。
```bash
$ xsartt pattern --help
根据正则模板和正则项集构建一个正则表达式.for example:
>> xsartt pattern -p '<~json_object~>' -k '{"value":"\\w+","key":"\\w+","json_object":"\\{\"<~key~>\":\"<~value~>\"\\}"}' -g key -g value
\{"(?P<key>\w+)":"(?P<value>\w+)"\}

Usage: xsartt.exe pattern [OPTIONS] --xpattern-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>

Options:
      --WTF <文件名>
          输出到文件
  -p, --xpattern-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>
          包含正则项的文本模板
  -k, --xpattern-dict <json格式文本 或 @(json/yaml/toml/xconfig)文件名 或 :xconfig数据路径, 可缺省为{}>
          正则项集 [default: {}]
  -g, --xpattern-group <正则项名称>
          正则表达式中的组, 即正则项的名称
  -h, --help
          Print help
```

### map
当初的写这个工具的主要目的就是用这个功能来自动代码生成。主要思想是用<~`json数据引用`~>的块嵌入到模板里，<~~>块可以嵌套，`json数据引用`的规则是“$|@.”开头， "$"引用根数据， "@"引用块内的数据。`json数据引用`的后面跟jq语句，不允许有空格。再后面可以跟:format:(json|yaml|toml|xconfig)直接输入各种格式文本。如果后面跟' map_as:'那后面的文本又是一个新的模板，数据来自`json数据引用`后的数据。
```bash
$ xsartt map --help
把正则项集数据(也可以是任意类JSON的数据)映射到输入文本模板中去.for example:
>> xsartt map -k '{"json_object":{"key":"a","value":"b"}}' -i 'map json_object as "<~$.json_object map key as (<~@.key~>), value as (<~@.value~>)"~>'   
map json_object as "map key as (a), value as (b)"

Usage: xsartt.exe map [OPTIONS] --input-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>

Options:
      --WTF <文件名>
          输出到文件
  -k, --xpattern-dict <json格式文本 或 @(json/yaml/toml/xconfig)文件名 或 :xconfig数据路径, 可缺省为{}>
          正则项集 [default: {}]
  -i, --input-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>
          输入文本
  -h, --help
          Print help
```

>参考示例 [example/map](example/map/README.md)

### find 
从文本中按正则表达式查找。
```bash
$ xsartt find --help
用构建的正则表达是在输入文本中找出匹配的文本.for example:
>> xsartt find -i '{"a":"b"}'  -p '<~json_object~>' -k '{"value":"\\w+","key":"\\w+","json_object":"\\{\"<~key~>\":\"<~value~>\"\\}"}' -g key -g value  
[{"g0": "{\"a\":\"b\"}", "index": "0", "key": "a", "value": "b"}]
>> xsartt find -i '{"a":"b"}'  -p '<~json_object~>' -k '{"value":"\\w+","key":"\\w+","json_object":"\\{\"<~key~>\":\"<~value~>\"\\}"}' -s '"json_object"={"key"={},"value"={}}'
[{"json_object":{"key":"a","value":"b"}}]

Usage: xsartt.exe find [OPTIONS] --xpattern-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径> --input-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>

Options:
      --WTF <文件名>
          输出到文件
  -p, --xpattern-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>
          包含正则项的文本模板
  -i, --input-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>
          输入文本
  -k, --xpattern-dict <json格式文本 或 @(json/yaml/toml/xconfig)文件名 或 :xconfig数据路径, 可缺省为{}>
          正则项集 [default: {}]
  -g, --xpattern-group <正则项名称>
          正则表达式中的组, 即正则项的名称
  -s, --xpattern-skeleton <toml格式文本 或 @(json/yaml/toml/xconfig)文件名 或 :xconfig数据路径>>
          描述正则表达式结构的类JSON的数据
  -h, --help
          Print help
```

### replace 
从文本中按正则表达式查找并替换文本。
```bash
$ xsartt replace --help
根据正则表达替换文本.for example:
>> xsartt replace -i '{"a":"b"}'  -p '<~json_object~>' -k '{"value":"\\w+","key":"\\w+","json_object":"\\{\"<~key~>\":\"<~value~>\"\\}"}' -g key -g value -r 'value is "$value", key is "$key"'
value is "b", key is "a"

Usage: xsartt.exe replace [OPTIONS] --xpattern-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径> --input-text <文本 或 @文件名 或 @-/*管道输 
入*/ 或 :xconfig数据路径> --replace-format <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>

Options:
      --WTF <文件名>
          输出到文件
  -p, --xpattern-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>
          包含正则项的文本模板
  -i, --input-text <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>
          输入文本
  -k, --xpattern-dict <json格式文本 或 @(json/yaml/toml/xconfig)文件名 或 :xconfig数据路径, 可缺省为{}>
          正则项集 [default: {}]
  -g, --xpattern-group <正则项名称>
          正则表达式中的组, 即正则项的名称
  -r, --replace-format <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>
          需要替换的格式, 可以用$<group>来表示引用正则表达式中的组.
  -w, --replace-write-back
          替换后直接会写回文件
  -h, --help
          Print help
```

### xconfig 
直接输出输出的输入的数据, 可选格式和路径(可用","分隔输出多个路径，合并为队列)。
```bash
$ xsartt xconfig --help
输出xconfig数据.for example:
>> xsartt -E -A 'a="b"' xconfig -O json
{"a":"b"}

Usage: xsartt.exe xconfig [OPTIONS] [path]

Arguments:
  [path]  xconfig数据的路径

Options:
      --WTF <文件名>                        输出到文件
      --freeze                           锁定xconfig为BASE64
  -O, --out-format <xconfig_out_format>  xconfig数据的输出格式, 可以为json/yaml/toml, 缺省为xconfig格式
  -h, --help                             Print help
```

### xshell 
调用OS的的可执行程序
```bash
$ xsartt xshell --help
使用xconfig中的数据(引用格式为\\<:$.'xconfig数据路径':\\>)作为参数执行命令行指令.for example:
>> xsartt xshell date
Thu, Mar 14, 2024  9:10:13 AM

Usage: xsartt.exe xshell [OPTIONS] <cmd>...

Arguments:
  <cmd>...  运行的命令行指令

Options:
      --WTF <文件名>     输出到文件
  -I, --ignore-error  忽略xshell/xmake失败。返回错误信息。
  -C, --cwd <目录名>     子命令运行基于的目录
  -h, --help          Print help
```

### xmake
用xconfig定义依赖关系(一个包含'TARGET:str', 'DEPENDENCY:list(str)', 'COMMAND:str'三个属性的对象的队列）， 按这个依赖关系构建。 和make差不多（实际上是调用make,所以使用前需按照GNU make)。
```bash
$ xsartt xmake --help
使用写在xconfig数据中XMAKE生成目标(依赖make).for example:
>> xsartt -A 'XMAKE_DEPENDENCY=[]' -A 'XMAKE=[{TARGET="t1",COMMAND="date"}]' xmake t1
Thu, Mar 14, 2024  9:14:19 AM


Usage: xsartt.exe xmake [OPTIONS] [target]...

Arguments:
  [target]...  xmake的目标

Options:
      --WTF <文件名>     输出到文件
  -I, --ignore-error  忽略xshell/xmake失败。返回错误信息。
  -C, --cwd <目录名>     子命令运行基于的目录
  -h, --help          Print help
```
>参考示例[example/xmake](example/xmake/README.md)

### eb64
主要目的是用来传递包含转义字符的文本。
```bash
$ xsartt eb64 --help
文本转base64.


Usage: xsartt.exe eb64

Options:
  -h, --help  Print help
```

### db64
eb64的反向操作。

### jaq
和jq用法差不多, 可使用'xsartt jaq -- --help'了解更多
```bash
$ xsartt jaq --help
由于经常用到jq, xsartt包裹一个rust开发的项目: [https://github.com/01mf02/jaq.git]

Usage: xsartt.exe jaq [OPTIONS] <jaq_args>...

Arguments:
  <jaq_args>...  jaq命令行选项, '-- --help' 获得帮助

Options:
      --WTF <文件名>  输出到文件
  -h, --help       Print help
```

### jsq
和jaq功能一样，但适合在xscript中使用。
```bash
$ xsartt jsq --help
简单jq，不支持复杂的命令行选项

Usage: xsartt.exe jsq [OPTIONS] --jq-filter <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>

Options:
      --WTF <文件名>
          输出到文件
  -j, --jq-filter <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>
          jq运算算法
  -k, --jq-input <json格式文本 或 @(json/yaml/toml/xconfig)文件名 或 :xconfig数据路径, 可缺省为{}>
          输入的json数据 [default: {}]
  -h, --help
          Print help
```

### xscript
一个脚本语言， 以上命令都可以在脚本语言中用到。
一些规则如下：
- 多行文本
采用 '...<<EOF\n' 开头， 'EOF\n' 结束。 好像还可以嵌套， 我不记得的了，呵呵。
如：
```xscript
echo <:<<EOF
line1
line2
:>
EOF
``` 

- 管道
定义了5个管道， 标号0-4和5-9管道相同， 0-4表示擦除后重新写入， 5-9表示在减去5后的那个管道添加写入。
每一行命令后跟'x|'表示写入管道x, x没有表示写入管道0.
读取管道在各条命令中和读取文件一样 '@-x',  x没有表示管道0。 如果管道中数据有固定格式如JSON, 可以用'@-x.json'解释为JSON数据。
写入管道的输出不再在标准输出里输出。

- 长参数
    输入包含空格或转义字符的参数， 用<:xxx xxx:>包裹，相当于shell里的'xxx xxx'

- 可以和bash一样， 在脚本文件第一行写入 '#!xsartt xscript' 直接调用。

```bash
$ xsartt xscript --help
执行xsartt脚本.for example:
>> xsartt xscript -f 'xshell date'
Mon, Jan  6, 2025  2:22:50 PM

Usage: xsartt.exe xscript [OPTIONS] [xsartt_script_file]

Arguments:
  [xsartt_script_file]  xsartt脚本文件

Options:
      --WTF <文件名>                                             输出到文件
  -C, --cwd <目录名>                                             子命令运行基于的目录
  -f, --from-content <文本 或 @文件名 或 @-/*管道输入*/ 或 :xconfig数据路径>  文本输入
  -h, --help
```

## 数据的加载

xsartt支持多种序列化数据输入(json/toml/yaml/xml/xconfig)

  - xconfig的规则如下：
    - 每个执行xsartt的当前目录下必须有一个.xConfig 文件。
    - 如果当前执行目录的上一级目录有.xConfig文件，其中配置也会被读入，直到有一个.xConfig文件中包含'ISROOT=true'
    - 加载的顺序为从ISROOT的那级目录开始，依次加载子目录。
    - 每个配置可以在文本中用'^~\$.xxx~^'引用前面加载的数据， xxx是JQ语法。同一个.xConfig按字母顺序加载，所以需要被引用的数据应放在一个小于引用他的节点上。 如 D.VALUE="引用 ^~$.C.VALUE~^" 可以引用节点'C.VALUE', 反之不行。但上一级目录数据可以。
    - 每个配置可以在文本中用'<!xxxx!>'直接执行xscript脚本的返回值作为输入。如 whoami="<! xshell id -un !>"

    >参考示例 [example/example/xconfig/多级目录配置](example/xconfig/level_1_dir/level_2_dir_1/README.md)

    - #INCXCFG 和 EB64 

        在任何xconfig中， 可使用'#INCXCFG XXXX'加载一些配置。
        XXXX 可以是 @/path/to/.xConfig 加载一个文件（**注意不要循环引用**）。
        如果XXXX 不是以@开头， XXXX将被解释为EB64字符串，自动通过DB64还原为xconfig正文。
        EB64字符串可以通过如下命令获得： 'echo -n <\xconfig正文> | xsartt eb64'


    

- xsartt支持.env。 
在.env中的格式为"XSARTT__X__XX='XXXXX'"的数据被xsartt读入，解释为 X.XX=XXXXXX ，也就是说'__'解释为'.',解释后的内容需符合前面描述的xconfig正文规则。
'XXXXX'可以为文本如'"aaaa"', 也可以是toml格式数据如'[{"a"="1"},2]', 甚至可以来自文件如'@/path/to/.xConfig'
在命令行， 可以用'-E'屏蔽从.env读入数据。
>**一个节点的数据只能在一个配置中写完，不能多次配置**

>参考示例[example/example/xconfig/dotent](example/xconfig/dotenv/README.md)

- 在命令行， 用-A(--update-xconfig 可多次使用)记载数据。

- 在xscript中， 用'set [ -a <\node> ] ..../@<\datafile> ' 加载数据。

## 安装和使用
        - xsartt 
                可直接执行/bin/xsartt/xsartt.exe

        - pyxsartt
                python -m pip install /bin/pyxsartt/pyxsartt-0.5.2-cp313-cp313-win_amd64.whl
pysartt使用示例
```python
import pyxsartt
import json
# 版本
print("pyxsartt.xsartt_version():{}".format(pyxsartt.xsartt_version()))
# 加载数据
print(
        "json.loads(pyxsartt.xsartt_xconfig(\"astring=\\\"string1\\\"\"))[\"astring\"]:{}"
        .format(
                json.loads(pyxsartt.xsartt_xconfig("astring=\"string1\"", "json"))["astring"]
        )
)
# 执行xscript脚本
print(
        "pyxsartt.xsartt_xscript(\"xscript -f <:echo abc:>\"):{}"
        .format(
                pyxsartt.xsartt_xscript("xscript -f <:echo abc:>")
        )
)    
```

## 其他

被裁员了,也许以后不搞软件了。把这个工具上传，希望有人能用得到。
当前先上传BIN, 如有人愿意用，再考虑上传源码（开发语言 RUST)
当前只编译了stable-x86_64-pc-windows-msvc版本， pyxsartt 安装包 在python v3.13基础上编译。

### 联系我

mail: xiaohongkey@126.com
