# 搜索公交线路工具
    有几百个人的家庭住址和单位的通勤距离需要统计。
    于是想到用python selenium结合xsartt自动从百度地图查询.

## 前提条件：
    安装了python(>=v3.13)
    安装了Edge浏览器

## 输入
    在当前目录的.xConfig中输入
```toml
[[search_input]]
from = "爱法花园"
to = "徐家汇"

# 可以输入任意条路线
[[search_input]]
from = "..."
to = "..."
```
## 使用
    点击 xmake.bat 文件
    或 在命令行执行 xmake.bat
    第一次运行会自动配置环境

## 输出：
    - 在 窗口 和 '查询结果/result.txt' 输出如下(举例）：
```
...>.venv\Scripts\xsartt.exe xscript search.xscript
搜索公交路线 爱法花园 -> 徐家汇:
票价¥5 直达 地铁9号线
56分钟 | 24.5公里 | 步行680米
票价¥6 沪川线/浦东27路/790路 → 地铁9号线
58分钟 | 24.8公里 | 步行400米
票价¥7 浦东68路 → 地铁9号线
1小时5分钟 | 24.9公里 | 步行390米
票价¥7 曹路2路 → 地铁12号线 → 地铁10号线 → 地铁11号线
1小时27分钟 | 30.0公里 | 步行490米
票价¥6 630路 → 783路 → 926路
2小时1分钟 | 26.3公里 | 步行120米

c:\project\trySelenium\edge>pause
请按任意键继续. . .
```
    - 在 '查询结果/'目录下，为每条路线截图保存。

## 后续
    - 如果要改变输入方式和输出方式可用xsartt进一步处理
