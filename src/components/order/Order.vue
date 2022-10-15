<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 搜索区域 -->
      <el-row :gutter="20">
        <el-col :span="6">
          <el-input
            placeholder="请输入用户ID"
            v-model="queryInfo.user_id"
            clearable
            @clear="getOrderList()"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getOrderList()"
            ></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 订单数据列表 -->
      <el-table :data="orderList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column
          label="订单编号"
          prop="order_number"
          width="350px"
        ></el-table-column>
        <el-table-column
          label="订单价格"
          prop="order_price"
          width="150px"
        ></el-table-column>
        <el-table-column label="是否付款" prop="pay_status" width="150px">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === '1'"
              >已付款</el-tag
            >
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send" width="150px">
          <template slot-scope="scope">{{ scope.row.is_send }}</template>
        </el-table-column>
        <el-table-column label="下单时间" prop="create_time">
          <template slot-scope="scope">{{
            scope.row.create_time | dateFormat
          }}</template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showPriceDialog(scope.row.order_id)"
            ></el-button>
            <el-button
              type="success"
              icon="el-icon-location"
              size="mini"
              @click="showProgressDialog"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页符区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 修改地址对话框 -->
    <el-dialog
      title="修改价格"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogclosed"
    >
      <el-form
        ref="editFormRef"
        :model="editForm"
        :rules="editFormRules"
        label-width="100px"
      >
        <el-form-item label="订单价格" prop="order_price">
          <el-input v-model="editForm.order_price"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="-footer">
        <el-button @click="priceVisible = false">取 消</el-button>
        <el-button type="primary" @click="editPrice">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 展示物流进度对话框 -->
    <el-dialog title="物流进度" :visible.sync="progressVisible" width="50%">
      <!-- 时间线 -->
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in progressInfo"
          :key="index"
          :timestamp="activity.time"
        >
          {{ activity.context }}
        </el-timeline-item>
      </el-timeline>
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
        pagesize: 10,
        user_id: '',
      },
      total: 0,
      //订单数据列表
      orderList: [],
      //修改地址对话框显示与
      editDialogVisible: false,
      //修改价格表单
      editForm: {},
      cityData,
      //控制物流进度对话框的显示与隐藏
      progressVisible: false,
      progressInfo: [],
      //修改地址表单验证规则
      editFormRules: {
        order_price: [
          { required: true, message: '请输入价格', trigger: 'blur' },
        ],
      },
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    //获取订单数据列表
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo,
      })
      if (res.meta.status !== 200) {
        this.$message.error('获取订单数据失败！')
      }
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    //监听pagesize变化
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    //监听页面数据变化
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    //展示修改价格对话框
    async showPriceDialog(id) {
      const { data: res } = await this.$http.get('orders/' + id)
      if (res.meta.status !== 200) {
        this.$message.error('查询订单信息失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    //监听修改价格对话框关闭事件
    editDialogclosed() {
      this.$refs.editFormRef.resetFields()
    },
    //修改价格
    editPrice() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          'orders/' + this.editForm.order_id,
          {
            order_price: this.editForm.order_price,
          }
        )
        if (res.meta.status !== 201) {
          this.$message.error('修改价格失败')
        }
        this.editDialogVisible = false
        this.getOrderList()
        this.$message.success('修改价格成功')
      })
    },
    //展示物流进度对话框
    async showProgressDialog() {
      //物流信息接口有问题
      // const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      // if (res.meta.status !== 200) {
      //   this.$message.error('获取物流信息失败！')
      // }
      // this.progressInfo = res.data
      this.progressInfo = [
        {
          time: '2018-05-10 09:39:00',
          ftime: '2018-05-10 09:39:00',
          context: '已签收,感谢使用顺丰,期待再次为您服务',
          location: '',
        },
        {
          time: '2018-05-10 08:23:00',
          ftime: '2018-05-10 08:23:00',
          context:
            '[北京市]北京海淀育新小区营业点派件员 顺丰速运 95338正在为您派件',
          location: '',
        },
        {
          time: '2018-05-10 07:32:00',
          ftime: '2018-05-10 07:32:00',
          context: '快件到达 [北京海淀育新小区营业点]',
          location: '',
        },
        {
          time: '2018-05-10 02:03:00',
          ftime: '2018-05-10 02:03:00',
          context:
            '快件在[北京顺义集散中心]已装车,准备发往 [北京海淀育新小区营业点]',
          location: '',
        },
        {
          time: '2018-05-09 23:05:00',
          ftime: '2018-05-09 23:05:00',
          context: '快件到达 [北京顺义集散中心]',
          location: '',
        },
        {
          time: '2018-05-09 21:21:00',
          ftime: '2018-05-09 21:21:00',
          context: '快件在[北京宝胜营业点]已装车,准备发往 [北京顺义集散中心]',
          location: '',
        },
        {
          time: '2018-05-09 13:07:00',
          ftime: '2018-05-09 13:07:00',
          context: '顺丰速运 已收取快件',
          location: '',
        },
        {
          time: '2018-05-09 12:25:03',
          ftime: '2018-05-09 12:25:03',
          context: '卖家发货',
          location: '',
        },
        {
          time: '2018-05-09 12:22:24',
          ftime: '2018-05-09 12:22:24',
          context: '您的订单将由HLA(北京海淀区清河中街店)门店安排发货。',
          location: '',
        },
        {
          time: '2018-05-08 21:36:04',
          ftime: '2018-05-08 21:36:04',
          context: '商品已经下单',
          location: '',
        },
      ]
      this.progressVisible = true
    },
  },
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
