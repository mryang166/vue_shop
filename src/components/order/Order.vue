<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 订单列表数据 -->
      <el-table :data="orderlist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column
          label="订单编号"
          prop="order_number"
          width="300px"
        ></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === '0'"
              >已付款</el-tag
            >
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send"></el-table-column>
        <el-table-column label="下单时间" prop="create_time" width="200px">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="70px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showBox(scope.row.order_id)"
            ></el-button>
            <!--  <el-button
              type="success"
              icon="el-icon-location"
              size="mini"
              @click="showProgressBox"
            ></el-button> -->
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- edit adress dialog -->
    <el-dialog
      title="修改地址"
      :visible.sync="addressVisible"
      width="50%"
      @close="addressDialogClosed"
    >
      <el-form
        :model="addressForm"
        :rules="addressFormRules"
        ref="addressFormRef"
        label-width="100px"
      >
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader
            :options="cityData"
            v-model="addressForm.address1"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary" @click="confirmAddress">确 定</el-button>
      </span>
    </el-dialog>
    <!-- show location progress -->
    <el-dialog title="物流进度" :visible.sync="progressVisible" width="50%">
      <span slot="footer" class="dialog-footer">
        <el-button @click="progressVisible = false">取 消</el-button>
        <el-button type="primary" @click="showProgressBox">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'

export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderlist: [],
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressFormRules: {
        address1: [
          { required: true, message: '请选择省市区/县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      // cityData:cityData,//重名可简写
      cityData,
      progressVisible: false,
      progressInfo: []
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败！')
      }
      // console.log(res)
      this.total = res.data.total
      this.orderlist = res.data.goods
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    // show edit adress dialog
    showBox(orderId) {
      // console.log(orderId)
      // console.log(this.orderlist)
      this.orderlist.forEach((item) => {
        if (item.order_id === orderId) {
          this.addressForm.address2 = item.consignee_addr
          window.sessionStorage.setItem('要修改的order_id', item.order_id)
          this.addressVisible = true
        }
      })
    },
    addressDialogClosed() {
      this.$refs.addressFormRef.resetFields()
    },
    confirmAddress() {
      this.$refs.addressFormRef.validate((valid) => {
        if (!valid) return this.$message.error('请填写地址')
        const editId = window.sessionStorage.getItem('要修改的order_id')
        // console.log(typeof(editId))
        this.orderlist.forEach((item) => {
          // console.log(item.order_id)
          // if(item.order_id == editId){
          if (item.order_id === parseInt(editId)) {
            // console.log("ok")
            const address =
              this.addressForm.address1 + this.addressForm.address2
            item.consignee_addr = address
            console.log(this.orderlist)
          }
        })
        this.addressVisible = false
      })
    },
    async showProgressBox() {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流信息失败！')
      }
      console.log(res)
      this.progressInfo = res.data
      this.progressVisible = true
      console.log(this.progressInfo)
    }
  }
}
</script>

<style lang="less" scoped></style>
