---
title: Echarts 添加堆叠工具选项
date: 2024-03-21 14:15:21
tags: ECharts
---

[https://echarts.apache.org/en/option.html#toolbox.feature.magicType](https://echarts.apache.org/en/option.html#toolbox.feature.magicType)

```js
option = {
    toolbox: {
        feature: {
            magicType: {
                type: ['line', 'bar', 'stack']
            }
        }
    },
    series: [
        {
            type: 'bar',
            data: [1, 2, 3, 4, 5, 6, 7]
        },
        {
            type: 'bar',
            data: [2, 3, 4, 5, 6, 7, 8]
        },
        {
            type: 'bar',
            data: [3, 4, 5, 6, 7, 8, 9]
        }
    ]
};
```

type 为 'stack' 时，会将所有的 bar 类型的 series 堆叠在一起。
