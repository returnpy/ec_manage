<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区 -->
    <el-card>
      <el-alert title="注意: 只允许为三级分类设置相关参数!" type="warning" show-icon :closable="false"></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类: </span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader popper-class="jiuwu" expand-trigger="hover" :options="catelist" :props="cateProps" v-model="selectedCateKeys" @change="handleChange">
          </el-cascader>
        </el-col>
      </el-row>

      <!-- tab 页签区 -->
      <el-tabs v-model="activeName" @click="handleTabClick">
        <!-- 添加动态参数的面板 -->
        <el-tab-pane
          label = "动态参数"
          name="many">
          <!--  添加参数按钮 -->
          <el-button type="primary" @click="addDialogVisible = true" size="mini" :disabled="isBtnDisabled">添加参数</el-button>
          <!-- 动态表格参数 -->
          <el-table :data="manyTableData" border stripe>
            <!--  展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染tag标签 -->
                <el-tag closable v-for="(item, index) in scope.row.attr_vals" :key="index" @close="handleClose(index, scope.row)">{{ item }}</el-tag>
                <!-- 输入的文本框 -->
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" placeholder="" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)"></el-input>
                <!-- 添加按钮 -->
                <el-button v-else class="button-new-tag" type="primary" @click="showInput(scope.row)" size="small">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" size="mini" @click="showEditDialog(scope.row.attr_id)" icon="el-icon-edit">编辑</el-button>
                <el-button type="danger" size="mini" @click="removeParams(scope.row.attr_id)" icon="el-icon-delete">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的面板 -->
        <el-tab-pane
          label = "静态属性"
          name="only">
          <el-button type="primary" @click="addDialogVisible = true" size="mini" :disabled="isBtnDisabled">添加属性</el-button>
          <!-- 静态表格属性 -->
          <el-table :data="onlyTableData" border stripe>
            <!--  展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染tag标签 -->
                <el-tag closable v-for="(item, index) in scope.row.attr_vals" :key="index" @close="handleClose(index, scope.row)">{{ item }}</el-tag>
                <!-- 输入的文本框 -->
                <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" placeholder="" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)"></el-input>
                <!-- 添加按钮 -->
                <el-button v-else class="button-new-tag" type="primary" @click="showInput(scope.row)" size="small">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" size="mini" @click="showEditDialog(scope.row.attr_id)" icon="el-icon-edit">编辑</el-button>
                <el-button type="danger" size="mini" @click="removeParams(scope.row.attr_id)" icon="el-icon-delete">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed">
      <!-- 添加参数的对话框 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改参数的对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed">
      <!-- 修改参数的对话框 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  props: {},
  data () {
    return {
      // 修改表单的验证规则对象
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      // 修改的表单数据对象
      editForm: {},
      // 控制修改对话框是否显示
      editDialogVisible: false,
      // 添加表单的验证规则对象
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      // 添加参数的表单数据对象
      addForm: {
        attr_name: ''
      },
      // 添加对话框是否显示
      addDialogVisible: false,
      // 动态参数的数据
      manyTableData: [],
      // 静态属性的数据
      onlyTableData: [],
      // 被激活的页签的名称
      activeName: 'many',
      // 商品分类列表
      catelist: [],
      // 级联选择框的配置对象
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 级联选择框双向绑定到数组
      selectedCateKeys: []
    }
  },
  computed: {
    // 动态计算标题名称
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    },
    // 按钮是否禁用
    isBtnDisabled () {
      if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类的 ID
    cateId () {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    }
  },
  created () {
    this.getCateList()
  },
  mounted () {},
  watch: {},
  methods: {
    // 持久化 attr_vals 的操作
    async saveAttrVals (row) {
      // 需要将数据持久化下来
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(',')
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数项失败!')
      }

      // 添加成功
      this.$message.success('修改参数项成功!')
    },
    // 删除对应的参数和选项
    handleClose (index, row) {
      row.attr_vals.splice(index, 1)
      this.saveAttrVals(row)
    },
    // 点击按钮展示输入框
    showInput (row) {
      row.inputVisible = true
      // 让文本框自动获得焦点
      // $nextTick 方法的作用 就是当页面元素被重新渲染之后,才会执行回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 文本框失去焦点或者按下enter
    async handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return null
      }
      // 如果能执行到这里 说明输入了内容
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 持久化
      this.saveAttrVals(row)
    },
    // 根据Ｉｄ删除对应的参数信息
    async removeParams (id) {
      const confirmResult = await this.$confirm('此操作将删除该参数，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      // 用户取消了删除的操作
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }

      // 删除的业务逻辑
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${id}`)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败!')
      }

      this.$message.success('删除参数成功！')
      this.getParamsData()
    },
    // 点击按钮修改参数信息
    editParams () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return null
        }
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败！')
        }

        this.$message.success('修改参数成功！')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 重置修改的表单
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    async showEditDialog (id) {
      // 查询当前参数的信息
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${id}`, {
        params: { attr_sel: this.activeName }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数信息失败!')
      }
      this.editForm = res.data
      // 点击按钮,展示修改对话框
      this.editDialogVisible = true
    },
    // 点击按钮 添加参数
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return null
        }

        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })

        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败!')
        }

        this.$message.success('添加参数成功!')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    // 监听添加对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // tab 页签点击事件的处理函数
    handleTabClick () {
      this.getParamsData()
    },
    // 级联选择框选中项变化触发
    async handleChange () {
      this.getParamsData()
    },
    // 获取参数列表的数据
    async getParamsData () {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return null
      }

      // 根据所选分类的 id 和当前所处的面板,获取对应的参数
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('参数列表获取失败!')
      }

      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(',') : []
        // 控制文本框的显示与隐藏
        item.inputVisible = false
        // 文本框中输入的值
        item.inputValue = ''
      })

      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 获取所有商品分类列表
    async getCateList () {
      const { data: res } = await this.$http.get('categories')

      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败!')
      }

      this.catelist = res.data
    }
  },
  components: {}
}
</script>

<style scoped lang="less">
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 150px;
}
</style>
