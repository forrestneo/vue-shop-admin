<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input :placeholder="$t('activity.coupon.phone')" v-model="listQuery.phone" style="width: 200px;" class="filter-item"/>
      <el-date-picker
        :default-time="['00:00:00', '23:59:59']"
        v-model="listQuery.timeRange"
        type="datetimerange"
        value-format="yyyy-MM-dd HH:mm:ss"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
        class="filter-item"/>
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">{{ $t('table.search') }}</el-button>
    </div>

    <el-table
      v-loading="listLoading"
      :key="tableKey"
      :data="list"
      stripe
      fit
      highlight-current-row
      style="width: 100%;">
      <el-table-column label="优惠券名称" align="center" width="200">
        <template slot-scope="scope">
          <span>{{ scope.row.templateName }}</span>
        </template>
      </el-table-column>
      <el-table-column label="手机号码" align="center" width="120">
        <template slot-scope="scope">
          <span>{{ scope.row.phone }}</span>
        </template>
      </el-table-column>
      <el-table-column label="优惠券类型" align="center" width="100">
        <template slot-scope="scope">
          <span>{{ getDiscountType(scope.row.templateType) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="状态" align="center" width="100">
        <template slot-scope="scope">
          <el-tag :type="scope.row.used | statusFilter">{{ scope.row.used ? '已使用' : '未使用' }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="满减订单金额" align="center" width="140">
        <template slot-scope="scope">
          <span class="icon-money"> <svg-icon icon-class="money" />{{ scope.row.orderAmount }}</span>
        </template>
      </el-table-column>
      <el-table-column label="优惠金额" align="center" width="100">
        <template slot-scope="scope">
          <span class="icon-money"> <svg-icon icon-class="money" />{{ scope.row.couponAmount != null ? scope.row.couponAmount : '——' }}</span>
        </template>
      </el-table-column>
      <el-table-column label="折扣力度" align="center" width="100">
        <template slot-scope="scope">
          <span>{{ scope.row.discountStrength != null ? scope.row.discountStrength + ' 折' : '——' }}</span>
        </template>
      </el-table-column>
      <el-table-column label="有效期" align="center" width="200">
        <template slot-scope="scope">
          <span>{{ scope.row.beginTime | parseTime('{y}-{m}-{d}') }} 至 {{ scope.row.endTime | parseTime('{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" align="center" width="200">
        <template slot-scope="scope">
          <span>{{ scope.row.createTime | parseTime('{y}-{m}-{d} {h}:{i}:{s}') }}</span>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.pageSize" @pagination="getCouponList" />
  </div>
</template>

<script>
import { postUserCoupons } from '@/api/activity/activity'
import waves from '@/directive/waves' // Waves directive
import Pagination from '@/components/Pagination' // Secondary package based on el-pagination

export default {
  name: 'UserCoupon',
  components: { Pagination },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        false: 'success',
        true: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        pageSize: 10,
        phone: undefined,
        timeRange: undefined
      },
      discountTypeOptions: [
        { key: 1, value: '优惠券' },
        { key: 2, value: '折扣券' }
      ]
    }
  },
  created() {
    this.getCouponList()
  },
  methods: {
    getCouponList() {
      this.listLoading = true
      postUserCoupons(this.listQuery).then(response => {
        this.list = response.data
        this.total = response.total

        setTimeout(() => {
          this.listLoading = false
        }, 1.5 * 1000)
      }).catch(() => {
        this.listLoading = false
      })
    },
    getDiscountType(type) {
      return this.discountTypeOptions.filter(v => v.key === type)[0].value
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getCouponList()
    }
  }
}
</script>
<style scoped>
  .icon-money {
    color: #f4516c;
  }
</style>
