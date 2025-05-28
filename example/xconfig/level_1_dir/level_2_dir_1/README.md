# example/xconfig 多级目录数据加载

这个示例演示了如何把数据如何在多级目录下加载。

- .xConfig
    包含数据的配置文件。
    其中当前目录的.xConfig不好含'ISROOT=true',所以会向上一级目录搜索，直到有一级目录下存在.xConfig并且存在'ISROOT=true'，则从此目录开始，逐个加载.xConfig.下级目录中的数据节点可以引用上一级目录的数据节点数据。
    同一个.xConfig中的数据节点也可以引用同文件中定义的数据节点数据，条件是节点名称的首字母小于被引用节点的节点名称首字母。

    具体规则可以看[README.md#xconfig](../../../../README.md#xconfig)

- show_config.sh
    显示数据。yaml格式。

