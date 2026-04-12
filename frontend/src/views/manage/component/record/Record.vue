
<template>
  <div class="record-container">
    <a-card :bordered="false">
      <div slot="title" style="display: flex; justify-content: space-between; align-items: center;">
        <span>正在派送的订单</span>
        <a-badge :count="deliveringOrders.length" :numberStyle="{ backgroundColor: '#52c41a' }" />
      </div>

      <a-spin :spinning="loading">
        <div v-if="deliveringOrders.length === 0 && !loading" class="empty-state">
          <a-empty description="暂无正在派送的订单" />
        </div>

        <a-list
          v-else
          item-layout="vertical"
          size="large"
          :pagination="{
            pageSize: 5,
            showTotal: (total) => `共 ${total} 个订单`,
            showSizeChanger: true,
            showQuickJumper: true
          }"
          :dataSource="deliveringOrders"
        >
          <a-list-item slot="renderItem" slot-scope="order" key="order.id">
            <a-list-item-meta>
              <div slot="title" style="display: flex; justify-content: space-between; align-items: center;">
                <span>
                  <a-icon type="shopping-cart" style="margin-right: 8px;" />
                  {{ order.code }}
                </span>
                <a-tag :color="getStatusColor(order.status)">
                  {{ getStatusText(order.status) }}
                </a-tag>
              </div>
              <div slot="description" style="margin-top: 8px;">
                <a-row :gutter="16">
                  <a-col :span="12">
                    <p><strong>客户：</strong>{{ order.userName }}</p>
                    <p><strong>联系电话：</strong>{{ order.contactMethod }}</p>
                    <p><strong>联系人：</strong>{{ order.contactPerson }}</p>
                  </a-col>
                  <a-col :span="12">
                    <p><strong>下单时间：</strong>{{ order.createDate }}</p>
                    <p><strong>支付时间：</strong>{{ order.payDate || '未支付' }}</p>
                    <p><strong>配送距离：</strong>{{ order.kilometre }} km</p>
                  </a-col>
                </a-row>
                <p style="margin-top: 8px;">
                  <strong>配送地址：</strong>
                  {{ order.province }}{{ order.city }}{{ order.area }}{{ order.address }}
                </p>
              </div>
            </a-list-item-meta>

            <div style="padding: 16px 0; border-top: 1px solid #f0f0f0;">
              <a-row :gutter="16">
                <a-col :span="8">
                  <div class="price-info">
                    <div class="price-label">订单金额</div>
                    <div class="price-value">¥{{ order.orderPrice }}</div>
                  </div>
                </a-col>
                <a-col :span="8">
                  <div class="price-info">
                    <div class="price-label">优惠金额</div>
                    <div class="price-value discount">-¥{{ order.discount }}</div>
                  </div>
                </a-col>
                <a-col :span="8">
                  <div class="price-info">
                    <div class="price-label">实付金额</div>
                    <div class="price-value final">¥{{ order.afterOrderPrice }}</div>
                  </div>
                </a-col>
              </a-row>
              <a-row :gutter="16" style="margin-top: 12px;">
                <a-col :span="12">
                  <div class="info-item">
                    <a-icon type="environment" />
                    <span>配送费：¥{{ order.distributionPrice }}</span>
                  </div>
                </a-col>
                <a-col :span="12">
                  <div class="info-item">
                    <a-icon type="gift" />
                    <span>获得积分：{{ order.integral }}</span>
                  </div>
                </a-col>
              </a-row>
            </div>

          </a-list-item>
        </a-list>
      </a-spin>
    </a-card>
  </div>
</template>

<script>import {mapState} from "vuex";

export default {
  name: "Record",
  computed: {
    ...mapState({
      currentUser: state => state.account.user
    }),
    deliveringOrders() {
      return this.orderList.filter(order => order.status === '2')
    }
  },
  data () {
    return {
      orderList: [],
      loading: false
    }
  },
  mounted () {
    this.queryOrderByStaff()
  },
  methods: {
    queryOrderByStaff () {
      this.loading = true
      this.$get('/cos/order-info/queryOrderByStaff', {
        staffId: this.currentUser.userId
      }).then(res => {
        if (res.data && res.data.data) {
          this.orderList = res.data.data
        }
      }).catch(err => {
        console.error('查询订单失败:', err)
        this.$message.error('查询订单失败')
      }).finally(() => {
        this.loading = false
      })
    },
    getStatusColor(status) {
      const statusMap = {
        '0': 'default',
        '1': 'orange',
        '2': 'blue',
        '3': 'green',
        '4': 'red',
        '5': 'purple'
      }
      return statusMap[status] || 'default'
    },
    getStatusText(status) {
      const statusMap = {
        '0': '待支付',
        '1': '已支付',
        '2': '配送中',
        '3': '已完成',
        '4': '已取消',
        '5': '退款中'
      }
      return statusMap[status] || '未知状态'
    },
    viewOrderDetail(order) {
      this.$emit('view-detail', order)
    },
    contactCustomer(order) {
      window.location.href = `tel:${order.contactMethod}`
    }
  },
}
</script>

<style scoped>.record-container {
  padding: 16px;
}

.empty-state {
  padding: 60px 0;
  text-align: center;
}

.price-info {
  text-align: center;
  padding: 12px;
  background-color: #fafafa;
  border-radius: 4px;
}

.price-label {
  font-size: 12px;
  color: #999;
  margin-bottom: 8px;
}

.price-value {
  font-size: 20px;
  font-weight: bold;
  color: #333;
}

.price-value.discount {
  color: #ff4d4f;
}

.price-value.final {
  color: #52c41a;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #666;
  font-size: 14px;
}

.info-item i {
  color: #1890ff;
}

/deep/ .ant-list-item {
  border-bottom: 1px solid #f0f0f0;
  transition: all 0.3s;
}

/deep/ .ant-list-item:hover {
  background-color: #fafafa;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
</style>
