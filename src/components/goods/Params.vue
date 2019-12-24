<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区域 -->
    <el-card>
      <el-alert :closable="false" show-icon title="注意：只允许为第三级分类设置相关参数" type="warning"></el-alert>
      <el-row>
        <el-col class="opt">
          <span>选择商品分类:</span>

          <!-- 级联选择框 -->
          <el-cascader
            v-model="value"
            :options=" cateList"
            :props="cateProps"
            expand-trigger="hover"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>

      <!-- tab标签 -->
      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isShow" @click="dialogVisible=true">动态参数</el-button>
          <!-- 表格 -->
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <!-- tag标签 -->
              <template slot-scope="scope">
                <el-tag
                  v-for="(item ,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="hongClose(i,scope.row)"
                >{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showCate(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="editDelete(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isShow" @click="dialogVisible=true">静态属性</el-button>
          <el-table :data="onlyTableData" border stripe>
            <el-table-column type="expand">
              <!-- tag标签 -->
              <template slot-scope="scope">
                <el-tag
                  v-for="(item ,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="hongClose(i,scope.row)"
                >{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showCate(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="editDelete(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 对话框 -->
    <el-dialog :title="'添加'+title" :visible.sync="dialogVisible" width="50%" @close="addClose">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label=" title" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addSure">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改对话框 -->
    <el-dialog :title="'修改'+title" :visible.sync="editlogVisible" width="50%" @close="editClose">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label=" title" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editlogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editSure">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      cateList: [],
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      value: [],
      // 页签
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      dialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入活动内容', trigger: 'blur' },
          { min: 1, max: 5, message: '长度在 1 到 5 个字符', trigger: 'blur' }
        ]
      },
      // 修改对话框
      editlogVisible: false,
      editForm: {},
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入活动内容', trigger: 'blur' },
          { min: 1, max: 5, message: '长度在 1 到 5 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表数据失败')
      }
      this.cateList = res.data
      // console.log(this.cateList)
    },
    // 级联选择框中的变化
    handleChange() {
      this.getParamsData()
    },
    // tab页签
    handleClick() {
      console.log(this.activeName)
      this.getParamsData()
    },
    // 获取参数数据
    async getParamsData() {
      // console.log(this.value);
      if (this.value.length !== 3) {
        this.value = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // console.log('tfrc')
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        { params: { sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败！')
      }
      console.log(res.data)
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制文本的显示与隐藏
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
    // 监听对话框关闭事件
    addClose() {
      this.$refs.addFormRef.resetFields()
    },
    addSure() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          { attr_name: this.addForm.attr_name, attr_sel: this.activeName }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('创建失败')
        }
        this.$message.success('创建成功')
        this.getParamsData()
        this.dialogVisible = false
      })
    },
    //  修改对话框
    async showCate(attr_id) {
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attr_id}`,
        { params: { attr_sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数信息失败')
      }
      this.editForm = res.data
      this.editlogVisible = true
    },
    editClose() {
      this.$refs.editFormRef.resetFields()
    },
    editSure() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数信息失败')
        }
        this.getParamsData()
        this.editlogVisible = false
      })
    },
    //  删除
    async editDelete(attr_id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attr_id}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    async handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveAttrVals(row)
    },
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('添加失败')
      }
      this.$message.success('修改参数成功')
    },
    showInput(row) {
      row.inputVisible = true
      // 让文本框自动获得焦点
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    hongClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },

  computed: {
    isShow() {
      if (this.value.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类id
    cateId() {
      if (this.value.length === 3) {
        return this.value[2]
      }
      return null
    },
    title() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
    // 控制tag标签输入的文本框
  }
}
</script>
<style scoped>
.opt {
  margin-top: 15px;
}
.el-tag {
  margin-left: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>