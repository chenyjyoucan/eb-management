<template>
  <div class="page">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table :data="orderlist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="order_number" label="订单编号"></el-table-column>
        <el-table-column prop="order_price" label="订单价格"></el-table-column>
        <el-table-column prop="pay_status" label="是否付款">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status == 1"
              >未付款</el-tag
            >
            <el-tag type="danger" v-else>已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货"></el-table-column>
        <el-table-column prop="create_time" label="下单时间">
          <template slot-scope="scope">
            {{ (scope.row.create_time * 1000) | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template>
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showBox"
            ></el-button>
            <el-button
              type="success"
              icon="el-icon-location"
              size="mini"
              @click="showProgressBox"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 20, 30]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <el-dialog title="修改地址" :visible.sync="addressVisible" width="40%">
      <el-form
        :model="addressForm"
        :rules="addressFormRules"
        ref="addressFormRef"
        label-width="100px"
      >
        <p style="text-align:center">没啥蛋用</p>
        <el-form-item label="省市区/县" prop="address1">
          <el-input v-model="addressForm.address1"></el-input>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <el-dialog title="物流进度" :visible.sync="progressVisible" width="40%">
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
export default {
  name: "",
  components: {},
  data() {
    return {
      addressVisible: false,
      progressVisible: false,
      progressInfo: [],
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderlist: [],
      addressForm: {
        address1: [],
        address2: ""
      },
      addressFormRules: {
        address1: [
          {
            required: true,
            message: "请选择省市区县",
            trigger: "blur"
          }
        ],
        address2: [
          {
            required: true,
            message: "请填写详细地址",
            trigger: "blur"
          }
        ]
      }
    };
  },
  created() {
    this.getOrderList();
  },
  methods: {
    async showProgressBox() {
      const { data: res } = await this.$http.get("/kuaidi/804909574412544580");
      if (res.meta.status !== 200) {
        this.$message({
          message: res.meta.msg,
          type: "error"
        });
        return;
      }
      this.progressInfo = res.data;

      this.progressVisible = true;
    },
    showBox() {
      this.addressVisible = true;
    },
    handleSizeChange(newsize) {
      this.queryInfo.pagesize = newsize;
      this.queryInfo.pagenum = 1;
      this.getOrderList();
    },
    handleCurrentChange(newpage) {
      this.queryInfo.pagenum = newpage;
      this.getOrderList();
    },
    async getOrderList() {
      const { data: res } = await this.$http.get("orders", {
        params: this.queryInfo
      });
      if (res.meta.status !== 200) {
        this.$message({
          message: res.meta.msg,
          type: "error"
        });
        return;
      }
      this.total = res.data.total;
      this.orderlist = res.data.goods;
    }
  }
};
</script>

<style scoped lang="less"></style>
