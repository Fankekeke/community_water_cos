
<template>
  <a-row :gutter="16" style="width: 100%;">
    <a-col :span="18">
      <div id="areas" class="map-container" style="height: 80vh;width: 100%"></div>
    </a-col>
    <a-col :span="6">
      <div class="staff-list-container">
        <h3 style="margin-bottom: 16px;">员工位置</h3>
        <a-spin :spinning="loading">
          <div v-if="staffList.length === 0 && !loading" style="text-align: center; color: #999; padding: 20px;">
            暂无员工位置信息
          </div>
          <div
            v-for="staff in staffList"
            :key="staff.id"
            class="staff-item"
            @click="locateStaff(staff)"
          >
            <a-avatar :src="getAvatarUrl(staff.images)" size="large" />
            <div class="staff-info">
              <div class="staff-name">{{ staff.name }}</div>
              <div class="staff-detail">
                <span>经度: {{ staff.lng }}</span>
                <span>纬度: {{ staff.lat }}</span>
              </div>
            </div>
          </div>
        </a-spin>
      </div>
    </a-col>
  </a-row>
</template>

<script>import {mapState} from 'vuex'

export default {
  name: 'Map',
  data () {
    return {
      map: null,
      staffList: [],
      loading: false,
      markers: []
    }
  },
  mounted () {
    setTimeout(() => {
      this.initMap()
    }, 500)
  },
  computed: {
    ...mapState({
      currentUser: state => state.account.user
    })
  },
  methods: {
    initMap () {
      this.map = new BMapGL.Map('areas')
      this.map.centerAndZoom(new BMapGL.Point(116.404, 39.915), 12)
      this.map.enableScrollWheelZoom(true)
      this.queryStaffMap()
    },
    queryStaffMap () {
      this.loading = true
      this.$get('/cos/staff-local-record/queryStaffByMerchant', {
        merchantId: this.currentUser.userId
      }).then(res => {
        let data = res.data.data
        if (data && Array.isArray(data)) {
          this.staffList = data
          this.addMarkersToMap(data)
        }
      }).finally(() => {
        this.loading = false
      })
    },
    addMarkersToMap (staffList) {
      this.clearMarkers()
      staffList.forEach(staff => {
        if (staff.lat && staff.lng) {
          const point = new BMapGL.Point(staff.lng, staff.lat)
          const marker = new BMapGL.Marker(point)

          const label = new BMapGL.Label(staff.name, {
            offset: new BMapGL.Size(20, -10)
          })
          marker.setLabel(label)

          const infoWindow = new BMapGL.InfoWindow(`            <div style="padding: 10px;">
              <p><strong>姓名：</strong>${staff.name}</p>
              <p><strong>编号：</strong>${staff.code}</p>
              <p><strong>性别：</strong>${staff.sex === 1 ? '男' : '女'}</p>
              <p><strong>状态：</strong>${staff.status === 1 ? '在职' : '离职'}</p>
              <p><strong>位置：</strong>经度 ${staff.lng}, 纬度 ${staff.lat}</p>
            </div>
          `)

          marker.addEventListener('click', () => {
            this.map.openInfoWindow(infoWindow, point)
          })

          this.map.addOverlay(marker)
          this.markers.push({ marker, staff })
        }
      })

      if (this.markers.length > 0) {
        this.fitMapView()
      }
    },
    clearMarkers () {
      this.markers.forEach(({ marker }) => {
        this.map.removeOverlay(marker)
      })
      this.markers = []
    },
    fitMapView () {
      if (this.markers.length === 0) return

      const points = this.markers.map(({ marker }) => marker.getPosition())
      const view = this.map.getViewport(points)
      this.map.centerAndZoom(view.center, view.zoom)
    },
    locateStaff (staff) {
      if (staff.lat && staff.lng) {
        const point = new BMapGL.Point(staff.lng, staff.lat)
        this.map.centerAndZoom(point, 16)

        const targetMarker = this.markers.find(m => m.staff.id === staff.id)
        if (targetMarker) {
          const event = new MouseEvent('click')
          targetMarker.marker.dispatchEvent(event)
        }
      }
    },
    getAvatarUrl (images) {
      return 'http://127.0.0.1:9527/imagesWeb/' + images.split(',')[0]
    }
  }
}
</script>

<style scoped>.staff-list-container {
  height: 80vh;
  overflow-y: auto;
  padding: 16px;
  background-color: #fff;
  border-radius: 4px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.staff-item {
  display: flex;
  align-items: center;
  padding: 12px;
  margin-bottom: 12px;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
  border: 1px solid #e8e8e8;
}

.staff-item:hover {
  background-color: #f5f5f5;
  border-color: #1890ff;
  box-shadow: 0 2px 8px rgba(24, 144, 255, 0.2);
}

.staff-info {
  margin-left: 12px;
  flex: 1;
}

.staff-name {
  font-size: 16px;
  font-weight: 500;
  color: #333;
  margin-bottom: 4px;
}

.staff-detail {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #999;
}

.staff-detail span {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
