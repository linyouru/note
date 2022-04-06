## cloud首页Echarts图表切换页面后不显示bug

### 背景

zws-cloud首页线上出现一个问题，用户登录后首页图表显示不出来，刷新页面后显示正常，但切换页面后图表又不能显示。且该现象只在线上环境出现，本地启动项目不能重现此现象

### 问题分析

##### 必现条件

测试此bug出现的必要条件发现该bug与刷新有关，用户第一次登录图表显示正常，切换页面也能正常显示；但若刷新页面后再切换页面则图表不能显示，重新登录也不能恢复，只有删除cookie才能恢复正常，一但刷新又将出现该bug。

##### 排查原因

在排除了渲染函数未执行、数据没获取到就渲染、缓存、cookie等原因后，查阅资料发现该bug与echarts实例id有关。

### 原因

- html

  ```html
  <div id="down-chart" class="down-chart" data-v-40d377f2="" style="width: 1500px; height: 400px; user-select: none; -webkit-tap-highlight-color: rgba(0, 0, 0, 0); position: relative;" _echarts_instance_="ec_1649236200409"></div>
  ```

  观察发现线上刷新页面后_echarts_instance_不会改变

- js

  ```js
  let onlineChart: any
  function createOnlineEcharts() {
      onlineChart ? onlineChart.dispose() : ''	//销毁了echarts实例
      type EChartsOption = echarts.EChartsOption
      const chartDom = document.getElementById('online-chart')!
      onlineChart = echarts.init(chartDom)
      let onlineOption: EChartsOption
      onlineOption = {
         //省略。。。
      }
      onlineOption && onlineChart.setOption(onlineOption)
  }
  ```

  刷新已经实例化的echarts图表时,echarts会先匹配改div容器上的_echarts_instance_属性值是否与实例对象的ID一样 如果一样则会在原有的结构上进行渲染,但上面已经销毁了实例，新的_echarts_instance_和div不一样，导致切换页面后图表显示不出来；

### 处理方法

- 方法1

  ```js
  let onlineChart: any
  function createOnlineEcharts() {
      onlineChart ? onlineChart.dispose() : ''	//销毁了echarts实例
      type EChartsOption = echarts.EChartsOption
      const chartDom = document.getElementById('online-chart')!
      chartDom.removeAttribute("_echarts_instance_") //删除dom的属性，重新生成
      onlineChart = echarts.init(chartDom)
      let onlineOption: EChartsOption
      onlineOption = {
         //省略。。。
      }
      onlineOption && onlineChart.setOption(onlineOption)
  }
  ```

  删除div的_echarts_instance_属性重新生成就可以正常显示了。

- 方法2

  方法1中每次数据改变都会销毁echarts实例再重新生成，这也是导致该bug的原因之一，至于为什么这么写是因为沿用了之前再tab标签页里渲染echarts的处理，因为官方给的说明:在图表容器被销毁之后，调用 echartsInstance.dispose 销毁实例；tab切换需要销毁实例，首页这里可以不用。

  ```js
  //不销毁echarts实例
  function createOnlineEcharts() {
      type EChartsOption = echarts.EChartsOption
      const chartDom = document.getElementById('online-chart')!
      let onlineChart = echarts.init(chartDom)
      let onlineOption: EChartsOption
      onlineOption = {
         //省略。。。
      }
      onlineOption && onlineChart.setOption(onlineOption)
  }
  ```

### 后记

*问题：*  

1、为什么相同的代码在本地div的_echarts_instance_会改变而线上不会？

2、为什么刷新才会触发这个bug？

后续有空再探究上述问题。。。

- 参考资料

  https://www.jianshu.com/p/fa77c365454f

  https://www.cnblogs.com/chenzibai/p/15927423.html

  



