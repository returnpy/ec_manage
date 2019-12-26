<template>
  <div>
    <!-- 面包屑导航 -->
    <ElBreadcrumb separator-class="el-icon-arrow-right">
      <ElBreadcrumbItem :to="{ path: '/home' }">首页</ElBreadcrumbItem>
      <ElBreadcrumbItem>权限管理</ElBreadcrumbItem>
      <ElBreadcrumbItem>权限列表</ElBreadcrumbItem>
    </ElBreadcrumb>

    <!-- 卡片视图 -->
    <ElCard>
      <el-table :data="rightsList" border stripe>
        <el-table-column
          type="index">
        </el-table-column>
        <el-table-column
          label="权限名称"
          prop="authName">
        </el-table-column>
        <el-table-column
          label="路径"
          prop="path">
        </el-table-column>
        <el-table-column
          label="权限等级"
          prop="level">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.level === '0'">一级</el-tag>
            <el-tag v-if="scope.row.level === '1'" type="success">二级</el-tag>
            <el-tag v-if="scope.row.level === '2'" type="warning">三级</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </ElCard>
  </div>
</template>

<script>
export default {
  props: {},
  data () {
    return {
      rightsList: []
    }
  },
  computed: {},
  created () {
    this.getRightsList()
  },
  mounted () {},
  watch: {},
  methods: {
    async getRightsList () {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败!')
      }
      this.rightsList = res.data
    }
  },
  components: {}
}
</script>

<style scoped lang="less"></style>
