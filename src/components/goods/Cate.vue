<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addSure">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        :show-index="true"
        border
        class="tree-table"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color:green"></i>
          <i class="el-icon-error" v-else style="color:red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mimn" v-if="scope.row.cat_level ===0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level ===1">二级</el-tag>
          <el-tag type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
        <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 3, 5, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类" :visible.sync="addDialogVisible" width="50%" @close="cateClose">
      <!-- 分类名称 -->
      <el-form ref="addformref" :model="addform" :rules="addformRules" label-width="100px">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addform.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!-- options数据源 -->
          <el-cascader
            expand-trigger="hover"
            :options="ParentList"
            :props="listCate"
            v-model="selectdKeys"
            @change="parentCateChange"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      cateList: [],
      total: 0,
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'hong'
        }
      ],
      // 添加分类
      addDialogVisible: false,
      addform: {
        cat_name: '',
        // 分类父 ID
        cat_pid: 0,
        // 分类层级
        cat_level: 0
      },
      addformRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ]
      },
      // 父级子类的数据
      ParentList: [],
      listCate: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectdKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据失败')
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 监听pagesize改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮弹出对话框
    addSure() {
      this.getParentList()
      this.addDialogVisible = true
    },
    async getParentList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据失败')
      }
      console.log(res.data)
      this.ParentList = res.data
    },
    parentCateChange() {
      console.log(this.selectdKeys)
      if (this.selectdKeys.length > 0) {
        this.addform.cat_pid = this.selectdKeys[this.selectdKeys.length - 1]
        this.addform.cat_level = this.selectdKeys.length
      } else {
        this.addform.cat_pid = 0
        this.addform.cat_level = 0
      }
    },
    addCate() {
      this.$refs.addformref.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addform)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addDialogVisible = false
      })
    },
    // 监听
    cateClose() {
      this.$refs.addformref.resetFields()
      this.selectdKeys = []
      this.addform.cat_level = 0
      this.addform.cat_pid = 0
    }
  }
}
</script>
<style scoped>
.tree-table {
  margin-top: 25px;
}
.el-cascader {
  width: 100%;
}
</style>