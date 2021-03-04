<!-- 商家销量统计的横向柱状图 -->
<template>
  <div class="com-container">
    <div class="com-chart" ref="seller_ref"></div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      chartsInstance:null,
      allData:null,  //服务器返回的数据
      currentPage:1,//当前显示页数
      totalPage:0, //一共有多少页
      timerId:null//对应定时器
    };
  },
  mounted(){
    this.initChart()
    this.getData()
    window.addEventListener('resize',this.screenAdapter)
    this.screenAdapter()//在页面加载完成时，主动进行屏幕的适配（如第一次进入页面时）
  },
  destroyed () {
    clearInterval(this.timerId)
    window.removeEventListener('resize',this.screenAdapter)//在组件销毁时，需要将监听器取消掉
  },
  methods: {
    //初始化echartsInstance对象,需要全局echarts对象
    initChart(){
      this.chartsInstance=this.$echarts.init(this.$refs.seller_ref,'chalk')
      //对图表初始化配置的控制
      const initOption = {
            title:{
         text:"🗝 商家销售统计",
         left:20,
         top:20
        },
        grid:{//坐标轴的配置
          top:'20%',
          left:'3%',
          right:'6%',
          bottom:'3%',
          containLabel:true //距离是包含坐标轴上的文字
        },
         xAxis:{
           type:'value'
         },
         yAxis:{
           type:'category'
         },
         tooltip:{//鼠标移上去的配置
          trigger:'axis',
          axisPointer: {
            type:'line',
            z:0,
            lineStyle:{
              color:'#2D3443'
            }
          }
         },
         series:[
           {
             type:"bar",
             label:{//文字的显示
               show:true,
               position:'right',
               textStyle:{
                 color:'white'
               }
             },
             itemStyle: {
              //  barBorderRadius:[0,20,20,0 ],//数组分别显示左上角，右上角，右下角，左下角的大小
               //指明颜色渐变的方向
               //指明不同百分比之下颜色的值
               color:new this.$echarts.graphic.LinearGradient(0,0,1,0,[
                 //百分之0状态之下的颜色值
                 {
                   offset:0,
                   color:'#5052ee'
                 },
                 //百分之100状态下的颜色值
                 {
                   offset:1,
                   color:'#ab6ee5'
                 }
               ])

             }
           }
         ]
      }
      this.chartsInstance.setOption(initOption)
      //对图表对象进行鼠标事件的监听
      this.chartsInstance.on('mouseover',()=>{
        clearInterval(this.timerId)
      })
      this.chartsInstance.on('mouseout',()=>{
        this.startInterval()
      })
    },
    //获取服务器的数据
    async getData(){
      //http://127.0.0.1:8888/api/seller,main.js配置的,开启koa_serve,node app.js
      const {data:ret}=await this.$http.get('seller')//是一个promise对象
      // console.log(ret)
      this.allData = ret
      //对数组排序
      this.allData.sort((a,b)=>{
        return a.value - b.value //从小到大的排序
      })
      //每五个元素显示一页
      this.totalPage = this.allData.length % 5 ===0 ? this.allData.length/5:this.allData.length/5+1
      //获取到数据之后更新图表
      this.updateChart()
      //启动定时器
      this.startInterval()
    },
    //更新图表
    updateChart () {
      //数据分页显示
      const start = (this.currentPage - 1) * 5
      const end = this.currentPage * 5
      const showData = this.allData.slice(start,end)
      const sellerNames = showData.map((item)=>{
        return item.name
      })
      const sellerValues = showData.map((item)=>{
        return item.value
      })
      const dataOption = {
         yAxis:{
           data:sellerNames
         },
         series:[
           {
             data:sellerValues,
           }
         ]
      }
      //将配置项设置给echarts实例对象
      this.chartsInstance.setOption(dataOption)
    },
    startInterval(){
      if(this.timerId){
        clearInterval(this.timerId)
      }
      this.timerId = setInterval(()=>{
        this.currentPage++
        if(this.currentPage>this.totalPage){//边界处理
          this.currentPage = 1
        }
        this.updateChart()
      },6000)
    },
    //当浏览器的大小发生变化时，会调用的方法，来完成屏幕的适配
    screenAdapter(){
      // console.log(this.$refs.seller_ref.offsetWidth)
      const titleFontSize = this.$refs.seller_ref.offsetWidth/100*3.6
      const adapterOption={//和分辨率大小相关的配置项
         title:{
         textStyle: {
           fontSize:titleFontSize
         }
        },
         tooltip:{
          axisPointer: {
            lineStyle:{
              width:titleFontSize
            }
          }
         },
         series:[
           {
             barWidth:titleFontSize, //控制图表的大小
             itemStyle: {
               barBorderRadius:[0,titleFontSize/2,titleFontSize/2,0 ],//数组分别显示左上角，右上角，右下角，左

             }
           }
         ]
      }
      this.chartsInstance.setOption(adapterOption)
      this.chartsInstance.resize()//手动的调用图表对象的resize才能产生效果
    }
  }
}
</script>
<style lang='less' scoped>
body{
  border:1px solid #ccc
}
 
</style>