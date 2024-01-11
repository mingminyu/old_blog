# 示例图

<show-structure depth="2"/>

## 1. 行为轨迹数据流向图

经常出差或旅行的人，行为轨迹经常表现为从一个城市到另外一个城市，我们想将用户行为轨迹图以数据流向图的方式绘制出来，并显示出每个地点所待天数以及开始日期，这样我们可以非常清晰地看到用户在空间上的行动喜好。

![数据流向图](%myimgs%/pyecharts_travel_streaming.png?raw=true)


```Python
from pyecharts.charts import Geo
from pyecharts import options as opts
from pyecharts.globals import ChartType, SymbolType
from pyecharts.render import make_snapshot
from snapshot_selenium import snapshot


# 定义图中节点显示明细信息
cities = [
    {"name": "广州", "value": 5, "data": "天数: 10\n日期: 2023-01-01"},
    {"name": "北京", "value": 10, "data": "天数: 10\n日期: 2023-01-01"},
    {"name": "杭州", "value": 30, "data": "天数: 10\n日期: 2023-01-01"},
    {"name": "重庆", "value": 15, "data": "天数: 10\n日期: 2023-01-01"},
]

# 定义样例数据，后面经纬度坐标只是标签，可替换为其他的数据
geo_cities_coords = {
    "广州": [113.27, 23.13],
    "北京": [116.4, 39.90],
    "杭州": [120.15, 30.28],
    "重庆": [106.55, 29.57],
}

# 获取第一个城市（可设置未常驻地）的坐标，让地图展现时一开始为此点为中心
first_city_coords = geo_cities_coords[cities[0]["name"]]

# 计算给的点的平均经纬度
city_coords = [geo_cities_coords[city["name"]] for city in cities]
avg_longitude, avg_latitude = np.array(city_coords).mean(axis=0)

# 创建 Geo 对象
c = (
    Geo(init_opts=opts.InitOpts(width="2000px", height="1200px"))
    .add_schema(
        maptype="china", 
        center=[avg_longitude, avg_latitude], 
        zoom=1.8
    )
    .set_series_opts(
        label_opts=opts.LabelOpts(is_show=False), 
        is_map_symbol_show=False
    )
    .set_global_opts(
        title_opts=opts.TitleOpts(title="Travel Streaming Graph"),
        # 调整 the visual map 范围
        # visualmap_opts=opts.VisualMapOpts(max_=100),
    )
)

# 画散点图，并添加标签
for city in cities:
    name = city["name"]
    value = city["value"]
    data = city["data"]

    coordinates = geo_cities_coords[name]
    c.add_coordinate(name, coordinates[0], coordinates[1])
    c.add(
        series_name=name,
        data_pair=[(name, value)],
        type_=ChartType.EFFECT_SCATTER,
        label_opts=opts.LabelOpts(
            formatter="{a} \n" + data,
            position="left",
            border_width=1,
            border_color="green",
            border_radius=2,
            color="#00008B",
            margin=5,
            rich={
                "align": "left",
            },
        ),
        itemstyle_opts=opts.ItemStyleOpts(
            color="#FF0000",
            border_color="#000000",
            border_width=1,
            opacity=0.8

        ),
        # 以所在天数画 point 大小，注意这个值不要设置超过 20，
        # 所以要做标准化 * 20
        symbol_size=value,
        color="#FF1493"
    )

c.add(
    "",
    [("广州", "北京"), ("广州", "杭州"), ("广州", "重庆")],
    type_=ChartType.LINES,
    label_opts=opts.LabelOpts(position="middle"),
    effect_opts=opts.EffectOpts(
        symbol=SymbolType.ARROW, symbol_size=6, color="blue"
    ),
    linestyle_opts=opts.LineStyleOpts(curve=0.2, color="#00CED1"),
)

# 生成HTML文件
# c.render("travel.html")

# 保存图片，需要安装 snapshot-selenium 库，且生成图片比较耗时
make_snapshot(snapshot, c.render(), "travel.png")
```
{collapsible="true" default-state="collapsed" collapsed-title="展开查看源码"}


城市坐标的获取也可以通过 PyECharts 内置的 JSON 数据来后去，如果想给节点添加 📍图标，可以参考以下示例代码。 

```Python
# 可以显示坐标
mark_points = [
    {"name": "广州", "coord": geo_cities_coords["广州"]},
    {"name": "北京", "coord": geo_cities_coords["北京"]},
    {"name": "杭州", "coord": geo_cities_coords["杭州"]},
    {"name": "重庆", "coord": geo_cities_coords["重庆"]},
]
c.set_series_opts(
    markpoint_opts=opts.MarkPointOpts(
        data=mark_points,
        symbol="pin",  # Use a pin symbol for the marked points
        label_opts=opts.LabelOpts(position="top"),
    )
)
```
{collapsible="false" default-state="expanded"}