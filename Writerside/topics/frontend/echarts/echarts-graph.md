# 关系图

<show-structure depth="2"/>

## 1. 构建关系图

## 2. 节点

## 3. 连接边

### 3.1 添加注释信息

在关系图中，通过指定 `links` 变量中的 `source` 和 `target` 键值为节点 ID 来建立连接边，如果想要给边添加注释信息，需要在 `links` 数据中再添加一个 `label` 属性，同时还需要配置 `edgeLabel` 变量来控制如何显示信息。示例代码如下:

```Javascript
option = {
    series: [{
        type: 'graph',
        layout: 'force',
        edgeLabel: {
            normal: {
                show: true,
                formatter: function (params) {
                    return params.data.label || '';
                },
            },
        },
        data: [
            { name: 'Node 1' }, { name: 'Node 2' }, { name: 'Node 3' }
        ],
        links: [
            { source: 'Node 1', target: 'Node 2', label: 'Link 1-2' }, 
            { source: 'Node 2', target: 'Node 3', label: 'Link 2-3' }
        ],
    }],
}
```

