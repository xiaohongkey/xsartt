# example/xmake

这个示例演示了如何使用xmake构建一个EXE, 这个EXE打印"hello, '用户名'!"。

- .xConfig
    包含数据的配置文件。
    - 'ISROOT=true'
        表示不会再向上一级目录引入数据

    - 'C_CODE'
        生成main.c的模板

    - [XMAKE]
        定义了构建的依赖关系和需要执行的动作。

- make.sh
    执行构建。

- make_clean.sh
    清理构建过程中的中间文件


**运行这个示例需要安装make, gcc.**
