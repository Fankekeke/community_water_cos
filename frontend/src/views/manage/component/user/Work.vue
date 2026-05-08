
<template>
  <div class="work-container">
    <div class="hero-section">
      <div class="hero-overlay">
        <div class="hero-content">
          <div class="welcome-title">
            <span class="title-icon">🛒</span>
            <span class="title-text">欢迎来到桶装水配送系统</span>
          </div>
          <div class="welcome-subtitle">新鲜 · 便捷 · 优惠 · 一站式购物体验</div>

          <div class="promo-badges">
            <div class="promo-badge">
              <a-icon type="check-circle" theme="filled" />
              <span>品质保证</span>
            </div>
            <div class="promo-badge">
              <a-icon type="thunderbolt" theme="filled" />
              <span>极速配送</span>
            </div>
            <div class="promo-badge">
              <a-icon type="gift" theme="filled" />
              <span>超值优惠</span>
            </div>
            <div class="promo-badge">
              <a-icon type="safety-certificate" theme="filled" />
              <span>安全放心</span>
            </div>
          </div>

          <div class="search-section">
            <a-card :bordered="false" class="search-card" hoverable>
              <a-row type="flex" align="middle" justify="center" class="search-row">
                <a-col :span="16">
                  <a-input
                    v-model="key"
                    placeholder="搜索商家名称..."
                    size="large"
                    class="search-input">
                    <a-icon slot="prefix" type="search" style="color: #999"/>
                  </a-input>
                </a-col>
                <a-col :span="4" :offset="1">
                  <a-button
                    type="primary"
                    size="large"
                    @click="fetch"
                    class="search-btn"
                    icon="search">
                    查找商家
                  </a-button>
                </a-col>
              </a-row>
            </a-card>
          </div>
        </div>
      </div>
    </div>

    <div class="features-section">
      <a-row :gutter="24" class="features-row">
        <a-col :span="6">
          <div class="feature-card">
            <div class="feature-icon" style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%)">
              <a-icon type="shopping" />
            </div>
            <div class="feature-content">
              <h3>海量商品</h3>
              <p>数万种优质商品供您选择</p>
            </div>
          </div>
        </a-col>
        <a-col :span="6">
          <div class="feature-card">
            <div class="feature-icon" style="background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%)">
              <a-icon type="clock-circle" />
            </div>
            <div class="feature-content">
              <h3>24小时营业</h3>
              <p>随时随地满足您的购物需求</p>
            </div>
          </div>
        </a-col>
        <a-col :span="6">
          <div class="feature-card">
            <div class="feature-icon" style="background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%)">
              <a-icon type="car" />
            </div>
            <div class="feature-content">
              <h3>快速配送</h3>
              <p>最快30分钟送达您手中</p>
            </div>
          </div>
        </a-col>
        <a-col :span="6">
          <div class="feature-card">
            <div class="feature-icon" style="background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%)">
              <a-icon type="like" />
            </div>
            <div class="feature-content">
              <h3>优质服务</h3>
              <p>专业团队为您提供贴心服务</p>
            </div>
          </div>
        </a-col>
      </a-row>
    </div>

    <div>
      <div id="areas" class="map-container" style="height: 500px;width: 100%"></div>
    </div>

    <div class="supermarkets-section">
      <div class="section-header">
        <h2><a-icon type="shop" /> 热门商家推荐</h2>
        <p>精选优质商家，为您提供最佳购物体验</p>
      </div>

      <a-row :gutter="24" class="supermarket-grid">
        <a-col :span="6" v-for="(item, index) in roomList" :key="index">
          <a-card
            :bordered="false"
            hoverable
            class="supermarket-card"
            @click="view(item)">
            <div class="card-image-wrapper">
              <a-carousel autoplay v-if="item.images !== undefined && item.images !== ''">
                <div class="carousel-item" v-for="(img, imgIndex) in item.images.split(',')" :key="imgIndex">
                  <img :src="'http://127.0.0.1:9527/imagesWeb/'+img" alt="超市图片">
                </div>
              </a-carousel>
              <div v-else class="no-image">
                <a-icon type="picture" />
                <p>暂无图片</p>
              </div>
              <div class="status-badge" :class="item.currentStatus === '1' ? 'open' : 'closed'">
                {{ item.currentStatus === '1' ? '营业中' : '休息中' }}
              </div>
            </div>

            <div class="card-content">
              <div class="supermarket-name">
                <a-icon type="shop" theme="filled" class="name-icon"/>
                <span>{{ item.name }}</span>
              </div>

              <div class="supermarket-info">
                <div class="info-item">
                  <a-icon type="environment" />
                  <span class="info-text">{{ item.address }}</span>
                </div>
                <div class="info-item">
                  <a-icon type="phone" />
                  <a-tag color="green">{{ item.phone }}</a-tag>
                </div>
              </div>

              <div class="card-actions">
                <a-button type="primary" block icon="shopping">
                  立即选购
                </a-button>
              </div>
            </div>
          </a-card>
        </a-col>
      </a-row>
    </div>

    <Map :orderData="orderMapView.merchantInfo"
         @close="handleorderMapViewClose"
         :orderShow="orderMapView.visiable">
    </Map>

  </div>
</template>

<script>
import {mapState} from 'vuex'
import VehicleView from './VehicleView.vue'
import Map from './Map.vue'
export default {
  name: 'Work',
  components: {Map, VehicleView},
  data () {
    return {
      bulletinAdd: {
        visiable: false,
        data: null
      },
      orderMapView: {
        merchantInfo: null,
        visiable: false
      },
      key: '',
      roomList: [],
      roomTypeList: [],
      loading: false,
      vehicleView: {
        visiable: false,
        data: null
      },
      rentView: {
        visiable: false,
        data: null
      },
      map: null,
      startDate: null,
      endDate: null
    }
  },
  computed: {
    ...mapState({
      currentUser: state => state.account.user
    })
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
      this.getWorkStatusList()
    },
    handleBulletinAddClose () {
      this.bulletinAdd.visiable = false
    },
    handleBulletinAddSuccess () {
      this.bulletinAdd.visiable = false
      this.$message.success('新增定制商品成功')
    },
    customAdd (row) {
      this.bulletinAdd.data = row
      this.bulletinAdd.visiable = true
    },
    handlevehicleViewClose () {
      this.vehicleView.visiable = false
    },
    handlevehicleViewSuccess () {
      this.vehicleView.visiable = false
      this.$message.success('添加订单成功')
    },
    orderSuccess () {
      this.rentView.visiable = false
      this.$message.success('添加订单成功')
      this.fetch()
    },
    collectDelete (id) {
      this.$delete(`/cos/collect-info/${id}`).then((r) => {
        this.$message.success('取消收藏成功')
        this.fetch()
      })
    },
    collectAdd (roomCode) {
      let data = { userId: this.currentUser.userId, roomCode }
      this.$post(`/cos/collect-info`, data).then((r) => {
        this.$message.success('成功')
        this.fetch()
      })
    },
    view (record) {
      this.orderMapView.merchantInfo = record
      this.orderMapView.visiable = true
    },
    handleorderMapViewClose () {
      this.orderMapView.visiable = false
    },
    getRoomType () {
      this.$get(`/cos/vehicle-type-info/list`).then((r) => {
        this.roomTypeList = r.data.data
      })
    },
    getWorkStatusList () {
      this.$get(`/cos/order-info/selectMerchantList`, { key: this.key }).then((r) => {
        this.roomList = r.data.data
        this.addMerchantMarkers()
      })
    },
    addMerchantMarkers () {
      if (!this.map || !this.roomList || this.roomList.length === 0) return

      this.map.clearOverlays()

      this.roomList.forEach(merchant => {
        if (merchant.longitude && merchant.latitude) {
          const point = new BMapGL.Point(merchant.longitude, merchant.latitude)

          const marker = new BMapGL.Marker(point)

          const label = new BMapGL.Label(merchant.name, {
            offset: new BMapGL.Size(20, -10)
          })
          label.setStyle({
            color: '#fff',
            backgroundColor: '#4285f4',
            border: '1px solid #4285f4',
            borderRadius: '4px',
            padding: '4px 8px',
            fontSize: '12px',
            fontWeight: 'bold'
          })
          marker.setLabel(label)

          const infoWindow = new BMapGL.InfoWindow(`
            <div style="padding: 10px; min-width: 250px;">
              <h4 style="margin: 0 0 10px 0; color: #4285f4;">${merchant.name}</h4>
              <p style="margin: 5px 0; font-size: 13px;"><strong>编号：</strong>${merchant.code}</p>
              <p style="margin: 5px 0; font-size: 13px;"><strong>地址：</strong>${merchant.address}</p>
              <p style="margin: 5px 0; font-size: 13px;"><strong>负责人：</strong>${merchant.principal || '- -'}</p>
              <p style="margin: 5px 0; font-size: 13px;"><strong>电话：</strong>${merchant.phone}</p>
              <p style="margin: 5px 0; font-size: 13px;"><strong>营业时间：</strong>${merchant.operateStartTime} - ${merchant.operateEndTime}</p>
              <p style="margin: 5px 0; font-size: 13px;"><strong>状态：</strong>${merchant.currentStatus === '1' ? '<span style="color: #34a853;">营业中</span>' : '<span style="color: #ea4335;">休息中</span>'}</p>
            </div>
          `)

          marker.addEventListener('click', () => {
            this.map.openInfoWindow(infoWindow, point)
          })

          marker.addEventListener('dblclick', () => {
            this.view(merchant)
          })

          this.map.addOverlay(marker)
        }
      })

      if (this.roomList.length > 0) {
        this.fitMapToMarkers()
      }
    },
    fitMapToMarkers () {
      const points = []
      this.roomList.forEach(merchant => {
        if (merchant.longitude && merchant.latitude) {
          points.push(new BMapGL.Point(merchant.longitude, merchant.latitude))
        }
      })

      if (points.length > 0) {
        const view = this.map.getViewport(points)
        const mapZoom = view.zoom
        const centerPoint = view.center
        this.map.centerAndZoom(centerPoint, mapZoom)
      }
    },
    fetch () {
      this.getWorkStatusList()
    }
  }
}
</script>

<style scoped lang="less">.work-container {
  background: #f0f2f5;
  min-height: calc(100vh - 64px);
}

.hero-section {
  height: 500px;
  background: url('/static/img/interior-3530343_1920.jpg') no-repeat center center;
  background-size: cover;
  position: relative;

  .hero-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    //background: linear-gradient(135deg, rgba(66, 133, 244, 0.85) 0%, rgba(66, 133, 244, 0.7) 50%, rgba(52, 168, 83, 0.75) 100%);
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 50px;
  }

  .hero-content {
    text-align: center;
    color: white;
    max-width: 900px;
    width: 100%;

    .welcome-title {
      font-size: 48px;
      font-weight: 700;
      margin-bottom: 15px;
      text-shadow: 2px 2px 8px rgba(0, 0, 0, 0.3);
      animation: fadeInDown 0.8s ease-out;

      .title-icon {
        margin-right: 15px;
        font-size: 52px;
      }

      .title-text {
        font-family: 'Microsoft YaHei', 'SimHei', sans-serif;
      }
    }

    .welcome-subtitle {
      font-size: 22px;
      font-weight: 400;
      margin-bottom: 30px;
      opacity: 0.95;
      letter-spacing: 3px;
      animation: fadeInUp 0.8s ease-out 0.2s both;
    }

    .promo-badges {
      display: flex;
      justify-content: center;
      gap: 30px;
      margin-bottom: 40px;
      animation: fadeInUp 0.8s ease-out 0.4s both;

      .promo-badge {
        display: flex;
        align-items: center;
        gap: 8px;
        background: rgba(255, 255, 255, 0.2);
        backdrop-filter: blur(10px);
        padding: 12px 20px;
        border-radius: 30px;
        font-size: 16px;
        font-weight: 500;
        transition: all 0.3s ease;

        &:hover {
          background: rgba(255, 255, 255, 0.3);
          transform: translateY(-3px);
          box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
        }

        i {
          font-size: 20px;
        }
      }
    }

    .search-section {
      animation: fadeInUp 0.8s ease-out 0.6s both;

      .search-card {
        background: rgba(255, 255, 255, 0.95);
        backdrop-filter: blur(10px);
        border-radius: 15px;
        box-shadow: 0 10px 40px rgba(0, 0, 0, 0.15);
        padding: 20px 30px;

        .search-row {
          .search-input {
            border-radius: 8px;
            transition: all 0.3s ease;

            &:hover, &:focus {
              box-shadow: 0 4px 12px rgba(66, 133, 244, 0.2);
            }
          }

          .search-btn {
            border-radius: 8px;
            height: 40px;
            font-weight: 500;
            background: linear-gradient(135deg, #4285f4 0%, #34a853 100%);
            border: none;
            transition: all 0.3s ease;

            &:hover {
              transform: translateY(-2px);
              box-shadow: 0 6px 20px rgba(66, 133, 244, 0.4);
            }
          }
        }
      }
    }
  }
}

.features-section {
  padding: 50px 30px;
  background: white;

  .features-row {
    max-width: 1200px;
    margin: 0 auto;

    .feature-card {
      display: flex;
      align-items: center;
      gap: 15px;
      padding: 25px;
      background: #fafafa;
      border-radius: 12px;
      transition: all 0.3s ease;
      cursor: pointer;

      &:hover {
        transform: translateY(-5px);
        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
        background: white;
      }

      .feature-icon {
        width: 60px;
        height: 60px;
        border-radius: 12px;
        display: flex;
        align-items: center;
        justify-content: center;
        color: white;
        font-size: 28px;
        flex-shrink: 0;
      }

      .feature-content {
        h3 {
          margin: 0 0 5px 0;
          font-size: 18px;
          font-weight: 600;
          color: #333;
        }

        p {
          margin: 0;
          font-size: 14px;
          color: #666;
        }
      }
    }
  }
}

.supermarkets-section {
  padding: 50px 30px;
  max-width: 1400px;
  margin: 0 auto;

  .section-header {
    text-align: center;
    margin-bottom: 40px;

    h2 {
      font-size: 32px;
      font-weight: 600;
      color: #333;
      margin: 0 0 10px 0;

      i {
        margin-right: 10px;
        color: #4285f4;
      }
    }

    p {
      font-size: 16px;
      color: #666;
      margin: 0;
    }
  }

  .supermarket-grid {
    .supermarket-card {
      border-radius: 12px;
      overflow: hidden;
      transition: all 0.3s ease;
      margin-bottom: 20px;
      cursor: pointer;

      &:hover {
        transform: translateY(-8px);
        box-shadow: 0 15px 40px rgba(0, 0, 0, 0.15);
      }

      .card-image-wrapper {
        position: relative;
        height: 200px;
        overflow: hidden;

        .carousel-item {
          height: 200px;

          img {
            width: 100%;
            height: 100%;
            object-fit: cover;
          }
        }

        .no-image {
          height: 200px;
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
          color: #999;

          i {
            font-size: 48px;
            margin-bottom: 10px;
          }

          p {
            margin: 0;
            font-size: 14px;
          }
        }

        .status-badge {
          position: absolute;
          top: 15px;
          right: 15px;
          padding: 6px 14px;
          border-radius: 20px;
          font-size: 13px;
          font-weight: 500;
          color: white;
          box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);

          &.open {
            background: linear-gradient(135deg, #34a853 0%, #0f9d58 100%);
          }

          &.closed {
            background: linear-gradient(135deg, #ea4335 0%, #c5221f 100%);
          }
        }
      }

      .card-content {
        padding: 20px;

        .supermarket-name {
          font-size: 18px;
          font-weight: 600;
          color: #333;
          margin-bottom: 12px;
          display: flex;
          align-items: center;
          gap: 8px;

          .name-icon {
            color: #4285f4;
          }
        }

        .supermarket-info {
          margin-bottom: 15px;

          .info-item {
            display: flex;
            align-items: center;
            gap: 8px;
            margin-bottom: 8px;
            font-size: 14px;
            color: #666;

            i {
              color: #999;
              font-size: 16px;
            }

            .info-text {
              flex: 1;
              overflow: hidden;
              text-overflow: ellipsis;
              white-space: nowrap;
            }
          }
        }

        .card-actions {
          margin-top: 15px;
        }
      }
    }
  }
}

@keyframes fadeInDown {
  from {
    opacity: 0;
    transform: translateY(-30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.ant-carousel ::v-deep .slick-slide {
  text-align: center;
  height: 200px;
  line-height: 200px;
  overflow: hidden;
}
</style>
