
<template>
  <div class="personal-container" style="width: 100%">
    <a-row :gutter="24" class="main-content">
      <a-col :xs="24" :sm="24" :md="16" :lg="12" :xl="10" :offset="7">
        <a-card :bordered="false" class="profile-card">
          <div slot="title" class="card-header">
            <a-icon type="user" class="header-icon" />
            <span class="header-title" style="color: black">个人信息设置</span>
          </div>

          <div class="avatar-section">
            <div class="avatar-upload-wrapper">
              <a-upload
                name="avatar"
                action="http://127.0.0.1:9527/file/fileUpload/"
                list-type="picture-card"
                :file-list="fileList"
                @preview="handlePreview"
                @change="picHandleChange"
              >
                <div v-if="fileList.length < 1">
                  <a-icon type="plus" />
                  <div class="ant-upload-text">
                    Upload
                  </div>
                </div>
              </a-upload>
              <a-modal :visible="previewVisible" :footer="null" @cancel="handleCancel">
                <img alt="example" style="width: 100%" :src="previewImage" />
              </a-modal>
            </div>
          </div>

          <a-modal :visible="previewVisible" :footer="null" @cancel="handleCancel" width="800px">
            <img alt="预览" style="width: 100%" :src="previewImage" />
          </a-modal>

          <a-form :form="form" layout="vertical" class="custom-form">
            <a-row :gutter="20">
              <a-col :span="24">
                <a-form-item label='员工姓名'>
                  <a-input
                    v-decorator="[
                      'name',
                      { rules: [{ required: true, message: '请输入姓名!' }] }
                    ]"
                    placeholder="请输入您的姓名"
                    size="large"
                    class="form-input"
                  >
                    <a-icon slot="prefix" type="user" class="input-icon" />
                  </a-input>
                </a-form-item>
              </a-col>

              <a-col :span="24">
                <a-form-item label='员工性别'>
                  <a-select
                    v-decorator="[
                      'sex',
                      { rules: [{ required: true, message: '请选择性别!' }] }
                    ]"
                    placeholder="请选择性别"
                    size="large"
                    class="form-select"
                  >
                    <a-select-option value="1">
                      <a-icon type="man" style="color: #1890ff; margin-right: 8px;" />男
                    </a-select-option>
                    <a-select-option value="2">
                      <a-icon type="woman" style="color: #eb2f96; margin-right: 8px;" />女
                    </a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>
            </a-row>

            <a-row :gutter="20" class="button-row">
              <a-col :span="24">
                <a-button
                  @click="handleSubmit"
                  type="primary"
                  :loading="loading"
                  size="large"
                  block
                  icon="save"
                  class="submit-btn"
                >
                  {{ loading ? '保存中...' : '保存修改' }}
                </a-button>
              </a-col>
            </a-row>
          </a-form>

          <drawerMap :childrenDrawerShow="childrenDrawer" @handlerClosed="handlerClosed"></drawerMap>
        </a-card>
      </a-col>
    </a-row>
  </div>
</template>

<script>import {mapState} from 'vuex'
import baiduMap from '@/utils/map/baiduMap'
import drawerMap from '@/utils/map/searchmap/drawerMap'
import moment from 'moment'
moment.locale('zh-cn')

const plainOptions = ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
const formItemLayout = {
  labelCol: {span: 24},
  wrapperCol: {span: 24}
}
function getBase64 (file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.readAsDataURL(file)
    reader.onload = () => resolve(reader.result)
    reader.onerror = error => reject(error)
  })
}
export default {
  name: 'merchant',
  data () {
    return {
      checkedList: [],
      indeterminate: true,
      checkAll: false,
      plainOptions,
      rowId: null,
      mapId: 'area',
      cardShow: false,
      localPoint: {},
      stayAddress: '',
      local: '',
      localData: [],
      formItemLayout,
      childrenDrawer: false,
      form: this.$form.createForm(this),
      userId: '',
      loading: false,
      fileList: [],
      previewVisible: false,
      previewImage: '',
      merchantInfo: null
    }
  },
  computed: {
    ...mapState({
      currentUser: state => state.account.user
    })
  },
  components: {
    drawerMap
  },
  mounted () {
    this.getmerchantByUser()
    baiduMap.initMap('areas')
  },
  methods: {
    moment,
    onChange (checkedList) {
      this.indeterminate = !!checkedList.length && checkedList.length < plainOptions.length
      this.checkAll = checkedList.length === plainOptions.length
      console.log(this.checkedList)
    },
    onCheckAllChange (e) {
      Object.assign(this, {
        checkedList: e.target.checked ? plainOptions : [],
        indeterminate: false,
        checkAll: e.target.checked
      })
    },
    getmerchantByUser () {
      this.$get('/cos/staff-info/queryStaffByUserId', { userId: this.currentUser.userId }).then((r) => {
        this.merchantInfo = r.data.data
        this.rowId = this.merchantInfo.id
        this.setFormValues(r.data.data)
      })
    },
    localhost (scenic) {
      baiduMap.clearOverlays()
      baiduMap.rMap().enableScrollWheelZoom(true)
      let point = new BMap.Point(scenic.longitude, scenic.latitude)
      baiduMap.pointAdd(point)
      baiduMap.findPoint(point, 16)
    },
    handleCancel () {
      this.previewVisible = false
    },
    async handlePreview (file) {
      if (!file.url && !file.preview) {
        file.preview = await getBase64(file.originFileObj)
      }
      this.previewImage = file.url || file.preview
      this.previewVisible = true
    },
    picHandleChange ({ fileList }) {
      this.fileList = fileList
    },
    handlerClosed (localPoint) {
      this.childrenDrawer = false
      if (localPoint !== null && localPoint !== undefined) {
        this.localPoint = localPoint
        let address = baiduMap.getAddress(localPoint)
        address.getLocation(localPoint, (rs) => {
          if (rs != null) {
            if (rs.address !== undefined && rs.address.length !== 0) {
              this.stayAddress = rs.address
            }
            if (rs.surroundingPois !== undefined) {
              if (rs.surroundingPois.address !== undefined && rs.surroundingPois.address.length !== 0) {
                this.stayAddress = rs.surroundingPois.address
              }
            }
            this.form.getFieldDecorator('address')
            let obj = {}
            obj['address'] = this.stayAddress
            this.form.setFieldsValue(obj)
          }
        })
      }
    },
    addPoint (point) {
      this.localPoint = point
    },
    onSearch () {
      let localData = []
      var options = {
        onSearchComplete: (results) => {
          if (local.getStatus() == BMAP_STATUS_SUCCESS) {
            for (var i = 0; i < results.getCurrentNumPois(); i++) {
              if (i === 0) {
                setTimeout(() => {
                  baiduMap.findPoint(results.getPoi(0).point, 15)
                }, 10)
              }
              localData.push(results.getPoi(i))
              if (results.getPoi(i).point !== undefined) {
                baiduMap.localPointAdd(results.getPoi(i))
              }
            }
            this.localData = localData
            this.cardShow = true
          }
        }
      }
      var local = new BMap.LocalSearch(baiduMap.rMap(), options)
      local.search(this.local)
    },
    onClose () {
      this.loading = false
      this.form.resetFields()
    },
    showChildrenDrawer () {
      this.childrenDrawer = true
    },
    onChildrenDrawerClose () {
      this.childrenDrawer = false
    },
    imagesInit (images) {
      if (images !== null && images !== '') {
        let imageList = []
        images.split(',').forEach((image, index) => {
          imageList.push({uid: index, name: image, status: 'done', url: 'http://127.0.0.1:9527/imagesWeb/' + image})
        })
        this.fileList = imageList
      }
    },
    setFormValues ({...user}) {
      this.userId = user.id
      let fields = ['name', 'status', 'sex', 'deptId', 'phone', 'baseSalary']
      Object.keys(user).forEach((key) => {
        if (key === 'images') {
          this.fileList = []
          this.imagesInit(user['images'])
        }
        if (key === 'sex' || key === 'status') {
          user[key] = user[key].toString()
        }
        if (fields.indexOf(key) !== -1) {
          this.form.getFieldDecorator(key)
          let obj = {}
          obj[key] = user[key]
          this.form.setFieldsValue(obj)
        }
      })
    },
    handleSubmit () {
      let images = []
      this.fileList.forEach(image => {
        if (image.response !== undefined) {
          images.push(image.response)
        } else {
          images.push(image.name)
        }
      })
      this.form.validateFields((err, values) => {
        if (!err) {
          this.loading = true
          let user = this.form.getFieldsValue()
          user.images = images.length > 0 ? images.join(',') : null
          user.id = this.rowId
          this.$put('/cos/staff-info', {
            ...user
          }).then((r) => {
            this.loading = false
            this.$message.success('修改成功')
          }).catch(() => {
            this.loading = false
          })
        }
      })
    }
  }
}
</script>

<style scoped>.personal-container {
  min-height: calc(100vh - 64px);
  padding: 40px 20px;
  position: relative;
  overflow: hidden;
}

.personal-container::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: radial-gradient(circle, rgba(255,255,255,0.1) 1px, transparent 1px);
  background-size: 50px 50px;
  animation: moveGrid 20s linear infinite;
  pointer-events: none;
}

@keyframes moveGrid {
  0% {
    transform: translate(0, 0);
  }
  100% {
    transform: translate(50px, 50px);
  }
}

.main-content {
  position: relative;
  z-index: 1;
}

.profile-card {
  border-radius: 16px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15);
  background: rgba(255, 255, 255, 0.98);
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
  overflow: hidden;
}

.profile-card:hover {
  box-shadow: 0 12px 48px rgba(0, 0, 0, 0.2);
  transform: translateY(-4px);
}

.card-header {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 0;
}

.header-icon {
  font-size: 28px;
  color: #667eea;
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
}

.header-title {
  font-size: 22px;
  font-weight: 600;
  color: black;
  -webkit-background-clip: text;
  background-clip: text;
}

.avatar-section {
  text-align: center;
  padding: 30px 0;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  margin: -24px -24px 24px -24px;
  border-bottom: 2px solid #e8e8e8;
}

.avatar-upload-wrapper {
  display: inline-block;
}

.avatar-uploader {
  width: 140px;
  height: 140px;
  margin: 0 auto;
  border-radius: 50%;
  overflow: hidden;
  box-shadow: 0 4px 16px rgba(102, 126, 234, 0.3);
  transition: all 0.3s ease;
  cursor: pointer;
}

.avatar-uploader:hover {
  box-shadow: 0 8px 24px rgba(102, 126, 234, 0.5);
  transform: scale(1.05);
}

.avatar-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: white;
  transition: all 0.3s ease;
}

.avatar-placeholder:hover {
}

.upload-icon {
  font-size: 40px;
  margin-bottom: 8px;
  animation: bounce 2s ease-in-out infinite;
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-8px);
  }
}

.upload-text {
  font-size: 13px;
  font-weight: 500;
}

.avatar-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.avatar-tip {
  margin-top: 12px;
  font-size: 13px;
  color: #666;
  font-weight: 500;
}

.custom-form {
  padding: 0 8px;
}

.custom-form >>> .ant-form-item-label > label {
  font-size: 15px;
  font-weight: 600;
  color: #333;
  margin-bottom: 8px;
}

.form-input,
.form-select {
  transition: all 0.3s ease;
}

.form-input >>> .ant-input,
.form-select >>> .ant-select-selector {
  border-radius: 8px;
  border: 2px solid #e8e8e8;
  transition: all 0.3s ease;
}

.form-input >>> .ant-input:focus,
.form-input >>> .ant-input-focused,
.form-select >>> .ant-select-focused .ant-select-selector {
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.15);
}

.input-icon {
  color: #667eea;
  font-size: 16px;
}

.button-row {
  margin-top: 32px;
}

.submit-btn {
  height: 48px;
  font-size: 16px;
  font-weight: 600;
  border-radius: 8px;
  border: none;
  box-shadow: 0 4px 16px rgba(102, 126, 234, 0.4);
  transition: all 0.3s ease;
  letter-spacing: 2px;
}

.submit-btn:hover {
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.6);
  transform: translateY(-2px);
}

.submit-btn:active {
  transform: translateY(0);
}

/deep/ .ant-card-head {
  border-bottom: 2px solid #f0f0f0;
  padding: 20px 24px;
  background: linear-gradient(to right, #fafafa, #fff);
}

/deep/ .ant-card-body {
  padding: 24px;
}

@media (max-width: 768px) {
  .personal-container {
    padding: 20px 12px;
  }

  .profile-card {
    margin: 0 -12px;
  }

  .avatar-section {
    padding: 20px 0;
  }

  .avatar-uploader {
    width: 120px;
    height: 120px;
  }

  .header-title {
    font-size: 18px;
  }
}
</style>
