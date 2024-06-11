<template>
  <div>
    <el-card shadow="never" class="border-1">
      <template #header>
        <div class="flex justify-between">
          <span class="text-sm">文章发布热点图</span>
        </div>
      </template>
      <!-- card body -->
      <div id="main" style="width: 100%; height: 300px"></div>
    </el-card>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, onBeforeMount } from "vue";
import * as echarts from "echarts";
import { getDashboardPublishArticleStatisticsInfo } from "@/api/admin/dashboard";

/*
 * origin：将整个年度的活跃图都显示出来，这就导致因屏幕尺寸不足难以显示后面月数的热图信息
 * repair：只显示已当前日期为基准，前四后二总共六个月的热图信息
//  * */
// const currentDate = new Date();
// const currentYear = currentDate.getFullYear(); //getFullYear()方法返回Date`对象的年份（四位数）。
// const currentMonth = currentDate.getMonth() + 1; //getMonth()方法返回Date`对象的月份（从0开始，即0代表1月，11代表12月）。因此，我们加1以使其变为常规的月份表示（1-12）。
// const startYear = currentYear; //起始年份为当前年份
// const startMonth = currentMonth - 4; // 获取最近三个月的数据
// const endYear = currentYear;
// const endMonth = currentMonth + 2;

const myData = [];
const ydata = [];
var myChart = null;

getDashboardPublishArticleStatisticsInfo().then((e) => {
  if (e.success) {
    let map = e.data;
    for (let key in map) {
      myData.push(key);
      ydata.push(map[key]);
    }
    console.log(ydata);
  }

  var chartDom = document.getElementById("main");
  myChart = echarts.init(chartDom);

  var option;

  // option = {
  //   visualMap: {
  //     show: false,
  //     min: 0,
  //     max: 10,
  //   },
  //   calendar: {
  //     range: [startYear + "-" + startMonth, endYear + "-" + endMonth],
  //   },
  //   series: {
  //     type: "heatmap",
  //     coordinateSystem: "calendar",
  //     data: myData,
  //   },
  //   gradientColor: ["#fff", "#40c463", "#30a14e", "#216e39"],
  // };

  option = {
    //你的代码
    tooltip: {
      trigger: "axis", //触发类型 柱状图
      axisPointer: { type: "shadow" }, //触发效果 移动上去 背景效果
    },
    legend: {
      show: true,
      right: 30,
      itemWidth: 12,
      itemHeight: 12,
      textStyle: {
        fontSize: 12,
        lineHeight: 12,
        rich: {
          a: {
            verticalAlign: "middle",
          },
        },
        padding: [0, 0, -2, 0], //[上、右、下、左]
      },
    },
    xAxis: [
      {
        splitLine: {
          show: false,
        },
        axisTick: {
          show: false,
        },
        axisLine: {
          show: true,
          lineStyle: {
            color: "rgba(0,0,0,0.5)",
          },
        },
        axisLabel: {
          color: "#979797",
          fontSize: 12,
        },
        type: "category",
        data: myData.slice(-30),
      },
    ],
    yAxis: {
      name: "",
      nameTextStyle: {
        color: "rgba(0,0,0,0.5)",
      },
      type: "value",
      min: 0,
      axisLine: {
        show: false,
      },
      axisLabel: {
        color: "#979797",
        fontSize: 12,
      },
      splitLine: {
        show: true,
        lineStyle: {
          type: "dashed",
          // color: '#2b2b2b'
        },
      },
      axisTick: {
        show: false,
      },
    },
    grid: {
      top: 50,
      left: 10,
      right: 10,
      bottom: 20,
      containLabel: true,
    },
    series: [
      {
        name: "每日发布量",
        type: "bar",
        barWidth: 12,
        // barGap:10,
        color: "#247FFF",
        data: ydata.slice(-30),
        itemStyle: {
          // borderRadius: 2,
          color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
            { offset: 0, color: "#FF8809FF" },
            { offset: 1, color: "#C04B002B" },
          ]),
        },
      },
    ],
  };

  option && myChart.setOption(option);
});
</script>
