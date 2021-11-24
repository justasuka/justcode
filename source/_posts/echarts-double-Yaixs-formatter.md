---
title: echarts-double-Yaixs-formatter
date: 2021-11-24 11:34:27
tags: ['echarts','vue']
---

## echart设置左右不同的刻度的Y轴的格式化

### 1. 需求:record_button:

1. 左侧Y轴是数值，右侧Y轴是百分比，或者单侧的格式为百分比
2. 过长的图分为两张但是图例同步操作

### 2. Code:desktop_computer:

+ 双侧设置不同的刻度格式:straight_ruler:

~~~javascript
this.schoolSettings = {
    showLine: ['proportion', 'rateOfReturn'],
    axisSite: { right: ['proportion', 'rateOfReturn'] },
    // 这里设置为`percent`就可以在label和yAxis中显示为百分比
    yAxisType: ['KMB', 'percent'],
    yAxisName: ['单位：亿元', '百分比'],
    labelMap: {
        'amount': '投资总额',
        'investmentIncome': '收益额',
        'proportion': '投资率',
        'rateOfReturn': '实际回报率'
    },
};
// extend
this.schoolExtend = {
    xAxis: {
        axisLabel: {
            interval: 0,
            rotate: 30,
            formatter: name => {
                return echarts.format.truncateText(name, 80, '12px Microsoft Yahei', '…')
            }
        },
        axisLine: { lineStyle: { color: "#fff", width: 1, } }
    },
    yAxis: {
        // 这里注释掉
        //                    axisLabel: {show: true, textStyle: {fontSize: "14px", color: "#fff",},interval:'auto',
        //						formatter: value => {
        //                        if (value > 1 || value === 0) return value;
        //                        else return value * 100 + '%'
        //                    }
        //                    },
        axisLine: {lineStyle: {color: "#fff", width: 1,}},
        splitLine: {show: true, lineStyle: {color: "rgba(9,233,255,0.1)", width: 1}}
    },
    legend: {
        show: true,
        textStyle: {
            color: "#e1e5d7"
        },
    },
    series:
    {
        barWidth: 20,
        itemStyle: {
            normal: {
                //								barBorderRadius: [5, 5, 5, 5],
                //								color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{offset: 0, color: "#5258cd"}, {
                //									offset: 0.5,
                //									color: "#8e9ae7"
                //								}, {offset: 1, color: "#abafff"}]),
            },
        },
    },
~~~

+ 单侧的设置百分比:pen:

~~~javascript
this.extend={
    yAxis: {
        axisLabel: {show: true, textStyle: {fontSize: "12px", color: "#fff",},interval:'auto',
        // 这里的格式化
        formatter: value => {
            return (value * 100).toFixed(2) + '%'
        }},
        axisLine: {lineStyle: {color: "#fff", width: 1,}},
        splitLine: {show: true, lineStyle: {color: "rgba(9,233,255,0.1)", width: 1}}
    },
}
~~~

[Echarts 官方文档](https://echarts.apache.org/zh/option.html#yAxis)

+ 同步图例

~~~html
<div class="item">
    <ve-histogram :data="chartData3" ref="h1" :legend="myLegend" :colors="color" height="37vh"
                  width="49vw" :settings="schoolSettings" :events="chartEvents" :extend="schoolExtend">
    </ve-histogram>
</div>
<div class="item" style="margin-top: -30px">
    <ve-histogram :data="chartData4" ref="h2" :legend="myLegend" :colors="color" height="37vh"
                  width="49vw" :settings="schoolSettings" :events="chartEvents" :extend="schoolExtend2">
    </ve-histogram>
</div>
~~~

重点是绑定`:legend`

`myLegend: {},`

点击事件

~~~javascript
var self = this
this.chartEvents = {
    click: function (e) {
        self.show = false;
		//  ...
    },
    // 同步图例
    legendselectchanged: (item) => {
        let currChartItem = self.$refs['h2']
        let selectedObj = item.selected
        self.$set(currChartItem['legend'], 'selected', selectedObj)
    },
};
// 隐藏图例
this.extend={
    legend: {
        show: false,
    },
}
~~~
