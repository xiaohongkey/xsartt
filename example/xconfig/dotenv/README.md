# example/xconfig/dotnet

这个示例演示了如何使用.env加载数据

- .xConfig
    包含数据的配置文件。
    - 'ISROOT=true'
        表示不会再向上一级目录引入数据


- .env
    从此文件中'XSARTT__'开头的配置中加载数据
    具体规则可以看[README.md#数据加载](../../../README.md#数据的加载)

- show_config.sh
    显示数据
