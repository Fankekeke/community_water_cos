
<template>
  <a-row :gutter="16" style="width: 100%;">
    <a-col :span="6">
      <div class="order-list-container">
        <div class="list-header">
          <h3><a-icon type="schedule" /> 配送订单列表</h3>
          <a-badge :count="orderList.length" :numberStyle="{ backgroundColor: '#52c41a' }" />
        </div>

        <a-spin :spinning="loading">
          <div v-if="orderList.length === 0 && !loading" class="empty-state">
            <a-empty description="暂无配送订单" />
          </div>

          <div
            v-for="(order, index) in sortedOrders"
            :key="order.id"
            class="order-card"
            :class="{ 'active-order': currentDeliveryIndex === index }"
            @click="selectOrder(order, index)"
          >
            <div class="order-header">
              <div class="order-sequence">
                <span class="sequence-number">{{ index + 1 }}</span>
              </div>
              <div class="order-basic">
                <div class="order-code">{{ order.code }}</div>
                <a-tag color="blue">配送中</a-tag>
              </div>
            </div>

            <div class="order-content">
              <div class="customer-info">
                <a-avatar :src="'http://127.0.0.1:9527/imagesWeb/' + order.userImages" size="small" />
                <div class="customer-detail">
                  <div class="customer-name">{{ order.userName }}</div>
                  <div class="customer-phone">{{ order.contactMethod }}</div>
                </div>
              </div>

              <div class="address-info">
                <a-icon type="environment" style="color: #ff4d4f; margin-right: 4px;" />
                <span class="address-text">{{ order.province }}{{ order.city }}{{ order.area }}</span>
              </div>

              <div class="order-stats">
                <div class="stat-item">
                  <a-icon type="car" />
                  <span>{{ order.kilometre }}km</span>
                </div>
                <div class="stat-item">
                  <a-icon type="pay-circle" />
                  <span>¥{{ order.afterOrderPrice }}</span>
                </div>
                <div class="stat-item">
                  <a-icon type="clock-circle" />
                  <span>{{ getDeliveryTime(order.kilometre) }}</span>
                </div>
              </div>
            </div>

            <div class="order-footer">
              <a-button type="link" size="small" @click.stop="navigateToOrder(order)">
                <a-icon type="navigation" />导航
              </a-button>
            </div>
          </div>
        </a-spin>
      </div>
    </a-col>

    <a-col :span="18">
      <div id="areas" class="map-container" style="height: 80vh;width: 100%"></div>
    </a-col>
  </a-row>
</template>

<script>import {mapState} from "vuex";

export default {
  name: "Order",
  computed: {
    ...mapState({
      currentUser: state => state.account.user
    }),
    sortedOrders() {
      return this.optimizedRoute
    }
  },
  data () {
    return {
      map: null,
      orderList: [],
      loading: false,
      optimizedRoute: [],
      markers: [],
      polyline: null,
      driving: null,
      currentDeliveryIndex: 0,
      staffLocation: null
    }
  },
  mounted () {
    setTimeout(() => {
      this.initMap()
    }, 500)
  },
  methods: {
    initMap () {
      this.map = new BMapGL.Map('areas')
      this.map.centerAndZoom(new BMapGL.Point(116.404, 39.915), 12)
      this.map.enableScrollWheelZoom(true)
      this.queryOrderByStaff()
    },

    queryOrderByStaff () {
      this.loading = true
      this.$get('/cos/order-info/queryOrderByStaff', {
        staffId: this.currentUser.userId
      }).then(res => {
        if (res.data && res.data.data) {
          this.orderList = res.data.data.filter(order => order.status === '2')
          this.optimizeDeliveryRoute()
          this.renderMapData()
        }
      }).catch(err => {
        console.error('查询订单失败:', err)
      }).finally(() => {
        this.loading = false
      })
    },

    optimizeDeliveryRoute() {
      if (this.orderList.length === 0) {
        this.optimizedRoute = []
        return
      }

      if (this.orderList.length === 1) {
        this.optimizedRoute = [...this.orderList]
        return
      }

      const orders = [...this.orderList]
      const optimized = []
      let currentPos = this.staffLocation || { lng: 116.404, lat: 39.915 }

      while (orders.length > 0) {
        let nearestIndex = 0
        let minDistance = this.calculateDistance(currentPos, orders[0])

        for (let i = 1; i < orders.length; i++) {
          const distance = this.calculateDistance(currentPos, orders[i])
          if (distance < minDistance) {
            minDistance = distance
            nearestIndex = i
          }
        }

        const nearestOrder = orders.splice(nearestIndex, 1)[0]
        optimized.push(nearestOrder)
        currentPos = { lng: nearestOrder.longitude, lat: nearestOrder.latitude }
      }

      this.optimizedRoute = optimized
    },

    calculateDistance(point1, point2) {
      const dx = point1.lng - point2.longitude
      const dy = point1.lat - point2.latitude
      return Math.sqrt(dx * dx + dy * dy)
    },

    renderMapData() {
      this.clearMapOverlays()

      if (this.optimizedRoute.length === 0) return

      const points = []

      this.optimizedRoute.forEach((order, index) => {
        const point = new BMapGL.Point(order.longitude, order.latitude)
        points.push(point)

        const marker = this.createOrderMarker(point, order, index + 1)
        this.map.addOverlay(marker)
        this.markers.push(marker)
      })

      if (points.length > 1) {
        this.drawPolyline(points)
        this.fitMapView(points)
      } else if (points.length === 1) {
        this.map.centerAndZoom(points[0], 15)
      }
    },

    addCustomMarker(point, order, sequence) {
      const html = `
        <div style="
          width: 36px;
          height: 36px;
          background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
          border-radius: 50%;
          display: flex;
          align-items: center;
          justify-content: center;
          color: white;
          font-weight: bold;
          font-size: 16px;
          box-shadow: 0 2px 8px rgba(0,0,0,0.3);
          border: 3px solid white;
          cursor: pointer;
        ">${sequence}</div>
      `

      const overlay = new BMapGL.CustomOverlay({
        position: point,
        content: html
      })

      overlay.addEventListener('click', () => {
        const infoWindow = new BMapGL.InfoWindow(`
          <div style="padding: 10px; min-width: 220px;">
            <h4 style="margin: 0 0 10px 0; color: #667eea;">订单 ${sequence}</h4>
            <p style="margin: 5px 0;"><strong>客户：</strong>${order.userName}</p>
            <p style="margin: 5px 0;"><strong>电话：</strong>${order.contactMethod}</p>
            <p style="margin: 5px 0;"><strong>联系人：</strong>${order.contactPerson}</p>
            <p style="margin: 5px 0;"><strong>地址：</strong>${order.address}</p>
            <p style="margin: 5px 0;"><strong>距离：</strong>${order.kilometre} km</p>
            <p style="margin: 5px 0;"><strong>金额：</strong>¥${order.afterOrderPrice}</p>
          </div>
        `)
        this.map.openInfoWindow(infoWindow, point)
      })

      this.map.addOverlay(overlay)
      this.markers.push(overlay)
    },

    createOrderMarker(point, order, sequence) {
      const marker = new BMapGL.Marker(point)

      const label = new BMapGL.Label(sequence.toString(), {
        offset: new BMapGL.Size(0, -35)
      })

      label.setStyle({
        color: '#fff',
        background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
        border: '3px solid #fff',
        borderRadius: '50%',
        width: '36px',
        height: '36px',
        padding: '0',
        fontSize: '16px',
        fontWeight: 'bold',
        boxShadow: '0 2px 8px rgba(0,0,0,0.3)',
        lineHeight: '36px',
        textAlign: 'center',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center'
      })

      marker.setLabel(label)

      const infoWindow = new BMapGL.InfoWindow(`
        <div style="padding: 10px; min-width: 220px;">
          <h4 style="margin: 0 0 10px 0; color: #667eea;">订单 ${sequence}</h4>
          <p style="margin: 5px 0;"><strong>客户：</strong>${order.userName}</p>
          <p style="margin: 5px 0;"><strong>电话：</strong>${order.contactMethod}</p>
          <p style="margin: 5px 0;"><strong>联系人：</strong>${order.contactPerson}</p>
          <p style="margin: 5px 0;"><strong>地址：</strong>${order.address}</p>
          <p style="margin: 5px 0;"><strong>距离：</strong>${order.kilometre} km</p>
          <p style="margin: 5px 0;"><strong>金额：</strong>¥${order.afterOrderPrice}</p>
        </div>
      `)

      marker.addEventListener('click', () => {
        this.map.openInfoWindow(infoWindow, point)
      })

      return marker
    },

    drawPolyline(points) {
      if (this.polyline) {
        this.map.removeOverlay(this.polyline)
      }

      this.polyline = new BMapGL.Polyline(points, {
        strokeColor: '#667eea',
        strokeWeight: 4,
        strokeOpacity: 0.8,
        strokeStyle: 'solid'
      })

      this.map.addOverlay(this.polyline)

      points.forEach((point, index) => {
        if (index < points.length - 1) {
          this.drawArrow(point, points[index + 1], index + 1)
        }
      })
    },

    drawArrow(startPoint, endPoint, sequence) {
      const arrow = new BMapGL.Polyline([startPoint, endPoint], {
        strokeColor: '#667eea',
        strokeWeight: 4,
        strokeOpacity: 0.8,
        icons: [new BMapGL.IconSequence(
          new BMapGL.Symbol(BMapGL.SymbolType.BMAP_SYMBOL_ARROW_FORWARD, {
            scale: 0.6,
            fillColor: '#667eea',
            fillOpacity: 1
          }),
          '30%',
          '70%'
        )]
      })
      this.map.addOverlay(arrow)
    },

    fitMapView(points) {
      const view = this.map.getViewport(points)
      this.map.centerAndZoom(view.center, view.zoom)
    },

    clearMapOverlays() {
      this.markers.forEach(marker => {
        this.map.removeOverlay(marker)
      })
      this.markers = []

      if (this.polyline) {
        this.map.removeOverlay(this.polyline)
        this.polyline = null
      }
    },

    selectOrder(order, index) {
      this.currentDeliveryIndex = index
      const point = new BMapGL.Point(order.longitude, order.latitude)
      this.map.centerAndZoom(point, 16)

      const marker = this.markers[index]
      if (marker) {
        const event = new MouseEvent('click')
        marker.dispatchEvent(event)
      }
    },

    viewOrderDetail(order) {
      this.$emit('view-detail', order)
    },

    navigateToOrder(order) {
      const destination = new BMapGL.Point(order.longitude, order.latitude)

      if (this.driving) {
        this.map.removeOverlay(this.driving)
      }

      this.driving = new BMapGL.DrivingRoute(this.map, {
        renderOptions: { map: this.map, autoViewport: true },
        onSearchComplete: (results) => {
          if (this.driving.getStatus() === BMAP_STATUS_SUCCESS) {
            this.$message.success('路线规划完成')
          }
        }
      })

      const startPoint = this.staffLocation
        ? new BMapGL.Point(this.staffLocation.lng, this.staffLocation.lat)
        : this.map.getCenter()

      this.driving.search(startPoint, destination)
    },

    getDeliveryTime(kilometre) {
      const avgSpeed = 30
      const time = (kilometre / avgSpeed) * 60
      return Math.ceil(time) + '分钟'
    }
  }
}
</script>

<style scoped>.order-list-container {
  height: 80vh;
  overflow-y: auto;
  padding: 16px;
  border-radius: 3px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
}

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 12px;
  border-bottom: 2px solid #e8e8e8;
}

.list-header h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: #333;
  display: flex;
  align-items: center;
  gap: 8px;
}

.empty-state {
  padding: 60px 0;
  text-align: center;
}

.order-card {
  background: white;
  border-radius: 3px;
  padding: 16px;
  margin-bottom: 16px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 2px solid transparent;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.order-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.25);
  border-color: #667eea;
}

.active-order {
  border-color: #667eea;
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.05) 0%, rgba(118, 75, 162, 0.05) 100%);
  box-shadow: 0 4px 16px rgba(102, 126, 234, 0.3);
}

.order-header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 12px;
}

.order-sequence {
  flex-shrink: 0;
}

.sequence-number {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-radius: 50%;
  font-weight: bold;
  font-size: 16px;
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.4);
}

.order-basic {
  flex: 1;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.order-code {
  font-size: 14px;
  font-weight: 600;
  color: #333;
}

.order-content {
  padding: 12px 0;
  border-top: 1px solid #f0f0f0;
  border-bottom: 1px solid #f0f0f0;
}

.customer-info {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 10px;
}

.customer-detail {
  flex: 1;
}

.customer-name {
  font-size: 15px;
  font-weight: 600;
  color: #333;
  margin-bottom: 2px;
}

.customer-phone {
  font-size: 12px;
  color: #999;
}

.address-info {
  display: flex;
  align-items: center;
  font-size: 13px;
  color: #666;
  margin-bottom: 10px;
}

.address-text {
  flex: 1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.order-stats {
  display: flex;
  justify-content: space-around;
  gap: 8px;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 12px;
  color: #666;
  background: #f5f5f5;
  padding: 4px 8px;
  border-radius: 4px;
  flex: 1;
  justify-content: center;
}

.stat-item i {
  color: #667eea;
}

.order-footer {
  display: flex;
  justify-content: space-between;
  margin-top: 12px;
}

/deep/ .ant-btn-link {
  padding: 4px 8px;
  color: #667eea;
}

/deep/ .ant-btn-link:hover {
  color: #764ba2;
}

.map-container {
  border-radius: 3px;
  overflow: hidden;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
}

@media (max-width: 1200px) {
  .order-list-container {
    height: auto;
    max-height: 80vh;
  }
}
</style>
