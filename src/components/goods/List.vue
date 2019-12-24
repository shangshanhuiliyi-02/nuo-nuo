<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" class="add" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <el-table :data="GoodsList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column label="商品价格(元)" prop="goods_price" width="100px"></el-table-column>
        <el-table-column label="商品重量" prop="goods_weight" width="70px"></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="140px">
          <template slot-scope="scope">{{scope.row.add_time | dateFormat}}</template>
        </el-table-column>
        <!-- 操作 -->
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="editGoods(scope.row.goods_id)"></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeById(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="this.queryInfo.pagenum"
        :page-sizes="[1, 5,15, 30]"
        :page-size="this.queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="this.total"
      ></el-pagination>
    </el-card>
    <!-- 编辑对话框 -->
    <el-dialog title="提示" :visible.sync="dialogVisible" width="50%" @close="editClose">
      <!-- 表单 -->
      <!-- <template slot-scope="scope"> -->
      <el-form :model="addForm" :rules="addRules" ref="addFormRef" label-width="100px">
  <el-form-item label="商品名称" prop="goods_name">
    <el-input v-model="addForm.goods_name"></el-input>
  </el-form-item>
  <el-form-item label="商品价格" prop="goods_price">
    <el-input v-model="addForm.goods_price"></el-input>
  </el-form-item>
  <el-form-item label="商品重量" prop="goods_weight">
    <el-input v-model="addForm.goods_weight"></el-input>
  </el-form-item>
      </el-form>
       
    <!-- </template> -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editSure">确 定</el-button>
      </span>
      
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      GoodsList: [],
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      // 编辑对话框
      dialogVisible:false,
      addForm:{
},
      addRules:{
        goods_name: [
            { required: true, message: '请输入商品名称', trigger: 'blur' },
            { min: 3, max: 100, message: '长度在 3 到 100 个字符', trigger: 'blur' }
          ],
           goods_price: [
            { required: true, message: '请输入商品价格', trigger: 'blur' },
            { min: 1, max: 5, message: '长度在 1 到 5 个字符', trigger: 'blur' }
          ],
           goods_weight: [
            { required: true, message: '请输入商品重量', trigger: 'blur' },
            { min: 2, max: 5, message: '长度在 1 到 5 个字符', trigger: 'blur' }
          ],
      }
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败')
      }
      this.$message.success('获取商品列表成功')

      this.GoodsList = res.data.goods
      this.total = res.data.total
      console.log(this.total)
    },
    // 分页开始
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    // 删除
    async removeById(goods_id) {
      const confirmRerult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmRerult !== 'confirm') {
        return this.$message.info('已取消了删除')
      }
      const { data: res } = await this.$http.delete(`goods/${goods_id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功')
      this.getGoodsList()
    },
    // 编辑
    async editGoods(goods_id){
      const {data :res} = await this.$http.get(`goods/${goods_id}`)
      if(res.meta.status !==200){
        return this.$message.error('获取失败')
      }
      this.addForm =res.data
      console.log(this.addForm)
      this.dialogVisible = true
    },
    editClose(){
      this.$refs.addFormRef.resetFields()
    },
    editSure(){
      this.$refs.addFormRef.validate(async valid =>{
        if(!valid) return
       const {data:res} = await this.$http.put(`goods/${this.addForm.goods_id}` , { 
         goods_name:this.addForm.goods_name,
         goods_price:this.addForm.goods_price,
         goods_number:this.addForm.goods_number,
         goods_weight:this.addForm.goods_weight})
       if(res.meta.status !==200){
         this.$message.error('修改失败')
       }
       this.$message.success('修改成功')
       this.getGoodsList()
       this.dialogVisible = false
       console.log(this.addForm);
       
      })
    },
    goAddpage(){
      this.$router.push('/good/add')
    }
  }
}
</script>
<style scoped>
.add {
  margin-left: 20px;
  
}
</style>