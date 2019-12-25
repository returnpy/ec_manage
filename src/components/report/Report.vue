<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区 -->
    <el-card>
      <!-- 2 为echarts准备一个具备大小的DOM   -->
      <div id="main" style="width: 100%; height: 500px;"></div>
    </el-card>
  </div>
</template>

<script>
// 1 导入 echarts
import echarts from 'echarts'
import _ from 'lodash'

export default {
  props: {},
  data () {
    return {
      // 需要合并的数据
      options: {
        title: {
          text: '用户来源'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#E9EEF3'
            }
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [
          {
            boundaryGap: false
          }
        ],
        yAxis: [
          {
            type: 'value'
          }
        ]
      }
    }
  },
  computed: {},
  created () {},
  async mounted () {
    // 3 基于准备好的ｄｏｍ，初始化ｅｃｈａｒｔｓ实例
    let myChart = echarts.init(document.querySelector('#main'))

    const { data: res } = await this.$http.get('reports/type/1')
    if (res.meta.status !== 200) {
      return this.$message.error('获取折线图数据失败!')
    }
    // 4 准备配置和数据项
    const result = _.merge(res.data, this.options)
    // 5 展示数据
    myChart.setOption(result)
  },
  watch: {},
  methods: {},
  components: {}
}
</script>

<style scoped lang="less"></style>
