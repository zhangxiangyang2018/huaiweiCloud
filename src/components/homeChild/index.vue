/*
 * @Author:  
 * @Date: 2018-11-14 09:00:50 
 * @Last Modified by: mikey.zhaopeng
 * @Last Modified time: 2019-03-21 16:20:48
 */
<template>
  <div class="first">
    <div class="drag">
      <el-select v-model="value"
                 placeholder="按区统计"
                 @change="getValue(value,value2)">
        <el-option v-for="(item,index) in options"
                   :key="index"
                   :label="item.label"
                   :value="item.value">
        </el-option>
      </el-select>
    </div>
    <div class="year">
      <el-date-picker v-model="dataValue2"
                      class="dragDataSelect"
                      type="year"
                      value-format="yyyy"
                      format="yyyy"
                      placeholder="请选择日期">
      </el-date-picker>
    </div>
    <el-button type="primary"
               round
               class="findBtn"
               @click="find">查看</el-button>
    <div id="chart1"
         v-show="oneShow"
         v-loading="loading1"
         :element-loading-text="loadSate1"
         element-loading-background="rgba(255, 255, 255, .5)"
         :element-loading-spinner="loadIcon"
         style="width:100%;height:500px"></div>
    <div v-show="divShow">
      <div id="chart5"
           v-loading="loading5"
           :element-loading-text="loadSate5"
           element-loading-background="rgba(255, 255, 255, .5)"
           :element-loading-spinner="loadIcon"
           style="width:100%;height:500px"></div>
      <div id="chart2"
           v-loading="loading2"
           :element-loading-text="loadSate2"
           element-loading-background="rgba(255, 255, 255, .5)"
           :element-loading-spinner="loadIcon"
           style="width:100%;height:500px"></div>

      <div id="chart3"
           v-loading="loading3"
           :element-loading-text="loadSate3"
           :element-loading-spinner="loadIcon"
           element-loading-background="rgba(255, 255, 255, .5)"
           style="width:100%;height:500px"></div>
      <div class="remoteState"
           style="width:100%;height:500px">
        <div id="chart4"
             v-loading="loading4"
             :element-loading-text="loadSate4"
             element-loading-background="rgba(255, 255, 255, .5)"
             :element-loading-spinner="loadIcon"
             style="width:100%;height:500px"></div>
        <div class="remoteDrag">
          <el-select v-model="value2"
                     placeholder="按PMS"
                     @change="getValueRemote(value,value2)">
            <el-option v-for="(item,index) in options2"
                       :key="index"
                       :label="item.label"
                       :value="item.value">
            </el-option>
          </el-select>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import Configport from "../../assets/js/address";
// console.log(Configport.netUrl);
var IP = Configport.netUrl;
import {
  indexChart1,
  indexChart2,
  indexChart3,
  indexChart4,
  indexChart5,
  indexChart6
} from "../../assets/js/drawEcharts.js";
let echarts = require("echarts/lib/echarts");
require("echarts/lib/chart/bar");
require("echarts/lib/chart/line");
export default {
  data() {
    return {
      region: "",
      dataValue2: "2018",
      loadSate1: "加载中...",
      loadSate2: "加载中...",
      loadSate3: "加载中...",
      loadSate4: "加载中...",
      loadSate5: "加载中...",
      loadIcon: "e",
      loading1: true,
      loading2: true,
      loading3: true,
      loading4: true,
      loading5: true,
      divShow: false,
      oneShow: true,
      chart1: "",
      chart2: "",
      chart3: "",
      chart4: "",
      chart5: "",
      optionObj: {},
      options: [],
      optionObj2: {},
      options2: [],
      value: "",
      label: "",
      value2: "GF1",
      legendName1: "", //推送卫星数据推送情况
      legendName11: "",
      legendName2: "", //终端接收卫星数据统计图
      legendName22: "",
      legendName3: "", //推送传感器数据情况统计
      legendName33: "",
      legendObj1: [],
      legendObj2: [],
      legendObj3: [],
      seriesData1: [],
      seriesData2: [],
      seriesData3: [],
      lineXdata: [],
      lineYdata1: [],
      lineYdata2: [],
      barXdata: [],
      barYdata1: [],
      barYdata2: [],
      barYdata3: [],
      barYdata4: [],
      barYdata5: [],
      barYdata6: []
    };
  },
  watch: {
    region(nl, ol) {
      // console.log(nl.ol);
      this.pms(nl);
    }
  },
  methods: {
    getValue(item, remote) {
      this.region = item;
      console.log(this.region);
    },
    getValueRemote(item, remote) {
      //默认PMS1
      this.value2 = remote;
      this.selectCity4(item, remote);
    },
    find() {
      if (this.value == "") {
        this.$notify.error({
          title: "请选择地区",
          onClick() {
            this.close();
          },
          duration: 900
        });
      } else {
        this.oneShow = false;
        this.divShow = true;
        this.loading1 = true;
        this.loading2 = true;
        this.loading3 = true;
        this.loading4 = true;
        this.loadSate1 = "加载中...";
        this.loadSate2 = "加载中...";
        this.loadSate3 = "加载中...";
        this.loadSate4 = "加载中...";

        // console.log(this.region, this.value2);
        this.selectCity1(this.region);
        this.selectCity2(this.region);
        this.selectCity3(this.region);
        this.selectCity4(this.region);
      }
    },
    echarts1() {
      //请求初始化的数据
      var _this = this;
      this.loading = true;
      this.axios({
        method: "get",
        url: IP + "/obsController/getCount",
        params: {
          year: "2018"
        }
      }).then(res => {
        console.log(res.data.data);
        for (var i in res.data.data) {
          _this.lineXdata.push(res.data.data[i].area);
          _this.lineYdata1.push(res.data.data[i].upNum);
          _this.lineYdata2.push(res.data.data[i].downNum);

          _this.barXdata.push(res.data.data[i].area);
          _this.barYdata1.push(res.data.data[i].upSize);
          _this.barYdata2.push(res.data.data[i].downSize);
        }
        indexChart1(
          _this.chart1,
          _this.lineXdata,
          _this.lineYdata1,
          _this.lineYdata2,
          _this.barYdata1,
          _this.barYdata2
        );
        if (
          _this.lineYdata1.every(ele => {
            return ele == undefined || ele == 0;
          }) &&
          _this.lineYdata2.every(ele => {
            return ele == undefined || ele == 0;
          }) &&
          _this.barYdata1.every(ele => {
            return ele == undefined || ele == 0;
          }) &&
          _this.barYdata2.every(ele => {
            return ele == undefined || ele == 0;
          })
        ) {
          _this.loading1 = true;

          _this.loadSate1 = "暂无数据";
          _this.loadIcon = "e";
        } else {
          this.loading1 = false;
        }

        // if (
        //   _this.barYdata1.every(ele => {
        //     return ele == undefined;
        //   }) &&
        //   _this.barYdata2.every(ele => {
        //     return ele == undefined;
        //   })
        // ) {
        //   _this.loading2 = true;
        //   _this.loadSate = "暂无数据";
        //   _this.loadIcon = "e";
        // } else {
        //   this.loading2 = false;
        // }
      });
      // .catch(err => {
      //   console.log("请求错误");
      // });
    },
    area() {
      //查询所有地区
      var _this = this;
      this.axios({
        method: "get",
        url: IP + "/obsController/getSelectArea"
      })
        .then(res => {
          console.log(res.data);
          for (var i in res.data) {
            _this.optionObj = {
              value: res.data[i].area,
              label: res.data[i].area
            };
            _this.options.push(_this.optionObj);
          }
        })
        .catch(err => {
          console.log("请求错误");
        });
      this.pms();
    },
    pms(region = "") {
      //选择卫星
      var _this = this;
      this.axios({
        method: "get",
        url:
          // "http://192.168.1.121:8090/obsController/getGF?area=" + _this.region
          IP + "/obsController/getGF?area=" + region
      })
        .then(res => {
          console.log(res);
          _this.options2 = [];
          for (var i in res.data.data) {
            _this.optionObj2 = {
              value: res.data.data[i],
              label: res.data.data[i]
            };
            _this.options2.push(_this.optionObj2);
          }
        })
        .catch(err => {
          console.log("请求错误");
        });
    },
    selectCity1(item) {
      //云端数据
      var _this = this;
      this.axios({
        method: "get",
        url: IP + "/obsController/statistics",
        params: {
          area: item,
          year: _this.dataValue2
        }
      })
        .then(res => {
          _this.barXdata = [];
          _this.barYdata1 = [];
          _this.barYdata2 = [];
          _this.barYdata3 = [];
          _this.barYdata4 = [];
          for (var i in res.data.data) {
            _this.barXdata.push(res.data.data[i].time);
            _this.barYdata1.push(res.data.data[i].receiveSize);
            _this.barYdata2.push(res.data.data[i].pushSize);
            _this.barYdata3.push(res.data.data[i].receiveNum);
            _this.barYdata4.push(res.data.data[i].pushNum);
          }
          _this.chart5 = echarts.init(document.getElementById("chart5"));
          indexChart3(
            _this.chart5,
            _this.barXdata,
            _this.barYdata1,
            _this.barYdata2,
            _this.barYdata3,
            _this.barYdata4
          );
          if (
            _this.barYdata1.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata2.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata3.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata4.every(ele => {
              return ele == undefined || ele == 0;
            })
          ) {
            _this.loading5 = true;
            _this.loadSate5 = "暂无数据";
            _this.loadIcon = "e";
            return;
          }
          _this.loading5 = false;
        })
        .catch(err => {
          console.log("请求错误");
        });
    },
    selectCity2(item) {
      //推送卫星数据情况统计
      var _this = this;
      this.axios({
        method: "get",
        url: IP + "/obsController/getUpPapar",
        params: {
          area: item,
          year: _this.dataValue2
        }
      })
        .then(res => {
          _this.barXdata = [];
          _this.seriesData1 = [];

          for (var i in res.data.data[0]) {
            //遍历图例
            _this.legendName1 = {
              name: res.data.data[0][i] + "数据量",
              icon: "circle"
            };
            _this.legendName11 = {
              name: res.data.data[0][i] + "数量",
              icon: "circle"
            };
            _this.legendObj1.push(_this.legendName1);
            _this.legendObj1.push(_this.legendName11);
          }
          for (var j in res.data.data[1]) {
            //遍历X轴
            _this.barXdata.push(res.data.data[1][j].dateTime);
          }
          for (var i in res.data.data[0]) {
            //获取series中的数据
            let data1 = [];
            let data2 = [];
            for (var j in res.data.data[1]) {
              data1.push(res.data.data[1][j]["counte" + (i * 1 + 1)]);
              data2.push(res.data.data[1][j]["sumSize" + (i * 1 + 1)]);
            }
            var item1 = {
              name: res.data.data[0][i] + "数量",
              type: "line",
              smooth: true,
              data: data1,
              yAxisIndex: 1
            };
            var item2 = {
              name: res.data.data[0][i] + "数据量",
              type: "bar",
              barMaxWidth: "30",
              data: data2,
              yAxisIndex: 0,
              itemStyle: {
                normal: {
                  barBorderRadius: [50, 50, 50, 50]
                }
              }
            };
            _this.seriesData1.push(item1);
            _this.seriesData1.push(item2);
          }
          _this.chart2 = echarts.init(document.getElementById("chart2"));
          indexChart4(
            _this.chart2,
            _this.barXdata,
            _this.legendObj1,
            _this.seriesData1
          );
          if (
            _this.barYdata1.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata2.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata3.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata4.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata5.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata6.every(ele => {
              return ele == undefined || ele == 0;
            })
          ) {
            _this.loading2 = true;
            _this.loadSate2 = "暂无数据";
            _this.loadIcon = "e";
            return;
          }
          _this.loading2 = false;
        })
        .catch(err => {
          console.log("请求错误");
        });
    },
    selectCity3(item) {
      var _this = this;
      //终端接收卫星数据统计图
      this.axios({
        method: "get",
        url: IP + "/obsController/getDownPapar",
        params: {
          area: item,
          year: _this.dataValue2
        }
      })
        .then(res => {
          _this.barXdata = [];
          _this.seriesData2 = [];

          for (var i in res.data.data[0]) {
            //遍历图例
            _this.legendName2 = {
              name: res.data.data[0][i] + "数据量",
              icon: "circle"
            };
            _this.legendName22 = {
              name: res.data.data[0][i] + "数量",
              icon: "circle"
            };
            _this.legendObj2.push(_this.legendName2);
            _this.legendObj2.push(_this.legendName22);
          }
          for (var j in res.data.data[1]) {
            //遍历X轴
            _this.barXdata.push(res.data.data[1][j].dateTime);
          }
          for (var i in res.data.data[0]) {
            //获取series中的数据
            let data1 = [];
            let data2 = [];
            for (var j in res.data.data[1]) {
              data1.push(res.data.data[1][j]["counte" + (i * 1 + 1)]);
              data2.push(res.data.data[1][j]["sumSize" + (i * 1 + 1)]);
            }
            var item1 = {
              name: res.data.data[0][i] + "数量",
              type: "line",
              smooth: true,
              data: data1,
              yAxisIndex: 1
            };
            var item2 = {
              name: res.data.data[0][i] + "数据量",
              type: "bar",
              barMaxWidth: "30",
              data: data2,
              yAxisIndex: 0,
              itemStyle: {
                normal: {
                  barBorderRadius: [50, 50, 50, 50]
                }
              }
            };
            _this.seriesData2.push(item1);
            _this.seriesData2.push(item2);
          }
          this.chart3 = echarts.init(document.getElementById("chart3"));

          indexChart5(
            _this.chart3,
            _this.barXdata,
            _this.legendObj2,
            _this.seriesData2
          );
          if (
            _this.barYdata1.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata2.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata3.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata4.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata5.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata6.every(ele => {
              return ele == undefined || ele == 0;
            })
          ) {
            _this.loading3 = true;
            _this.loadSate3 = "暂无数据";
            _this.loadIcon = "e";
            return;
          }
          _this.loading3 = false;
        })
        .catch(err => {
          console.log("请求错误");
        });
    },
    selectCity4(item, remote) {
      var _this = this;
      //推送传感器数据情况统计
      this.axios({
        method: "get",
        url: IP + "/obsController/getSensor",
        params: {
          area: _this.region,
          pms: _this.value2,
          year: _this.dataValue2
        }
      })
        .then(res => {
          _this.barXdata = [];
          _this.seriesData3 = [];
          console.log(res.data.data);
          for (var i in res.data.data[0]) {
            //遍历图例
            _this.legendName3 = {
              name: res.data.data[0][i] + "数据量",
              icon: "circle"
            };
            _this.legendName33 = {
              name: res.data.data[0][i] + "数量",
              icon: "circle"
            };
            _this.legendObj3.push(_this.legendName3);
            _this.legendObj3.push(_this.legendName33);
          }
          for (var j in res.data.data[1]) {
            //遍历X轴
            _this.barXdata.push(res.data.data[1][j].dateTime);
          }

          for (var i in res.data.data[0]) {
            //获取series中的数据
            let data1 = [];
            let data2 = [];
            for (var j in res.data.data[1]) {
              data1.push(res.data.data[1][j]["counteUp" + (i * 1 + 1)]);
              data2.push(res.data.data[1][j]["sumSizeUp" + (i * 1 + 1)]);
            }
            var item1 = {
              name: res.data.data[0][i] + "数量",
              type: "line",
              smooth: true,
              data: data1,
              yAxisIndex: 1
            };
            var item2 = {
              name: res.data.data[0][i] + "数据量",
              type: "bar",
              barMaxWidth: "30",
              data: data2,
              yAxisIndex: 0,
              itemStyle: {
                normal: {
                  barBorderRadius: [50, 50, 50, 50]
                }
              }
            };
            _this.seriesData3.push(item1);
            _this.seriesData3.push(item2);
          }
          this.chart4 = echarts.init(document.getElementById("chart4"));
          indexChart6(
            _this.chart4,
            _this.barXdata,
            _this.legendObj3,
            _this.seriesData3
          );
          if (
            _this.barYdata1.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata2.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata3.every(ele => {
              return ele == undefined || ele == 0;
            }) &&
            _this.barYdata4.every(ele => {
              return ele == undefined || ele == 0;
            })
          ) {
            _this.loading4 = true;
            _this.loadSate4 = "暂无数据";
            _this.loadIcon = "e";
            return;
          }
          _this.loading4 = false;
        })
        .catch(err => {
          console.log("请求错误");
        });
    },
    selfAdaption() {
      //图表自适应
      let _this = this;
      setTimeout(() => {
        window.addEventListener("resize", function() {
          _this.chart1.resize();
          _this.chart2.resize();
          _this.chart3.resize();
          _this.chart4.resize();
          _this.chart5.resize();
        });
      }, 10);
    }
  },
  mounted() {
    this.chart1 = echarts.init(document.getElementById("chart1"));
    this.echarts1();
    this.area();
    this.selfAdaption();
  }
};
</script>
<style lang="less" scoped>
.first {
  .drag {
    float: left;
    margin: 10px 0 20px;
    padding: 10px 15px;
    border-radius: 10px;
    color: #fff;
    .el-dropdown {
      color: #fff;
    }
  }
  .year {
    float: left;
    margin: 10px 0 20px;
    padding: 10px 15px;
    border-radius: 10px;
  }
  .findBtn {
    float: left;
    margin: 20px 0 0 40px;
  }
  .remoteState {
    float: left;
    background: #fff;
    border-radius: 15px;
    box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.3);
    position: relative;
    .remoteDrag {
      position: absolute;
      right: 20px;
      top: 20px;
    }
  }

  #chart1,
  #chart2,
  #chart3,
  #chart4,
  #chart5 {
    float: left;
    background: #fff;
    border-radius: 15px;
    box-shadow: 5px 5px 10px rgba(0, 0, 0, 0.3);
  }
  #chart1,
  #chart5 {
    margin-bottom: 80px;
  }
  #chart2,
  #chart3,
  #chart4 {
    margin-bottom: 30px;
  }
}
</style>
