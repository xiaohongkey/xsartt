# example/map

这个示例演示了如何把数据映射到模板中去。

- .xConfig
    包含数据的配置文件。
    - 'ISROOT=true'
        表示不会再向上一级目录引入数据

    - 需要映射的数据挂在'input'这个节点上

    具体规则可以看[README_cn.md#xconfig](../../README_cn.md#xconfig)

- map_template.txt
    模板， 其中<~xxx~>是需要被映射的部分。规则可以看[README_cn.md#map](../../README_cn.md#map)

- test.xscript
    执行映射的脚本文件

- test.sh
    在当前目录下调用test.xscript
