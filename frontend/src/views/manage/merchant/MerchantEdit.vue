<template>
  <a-drawer
    title="修改商家"
    :maskClosable="false"
    width=1000
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="show"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form :form="form" layout="vertical">
      <a-row :gutter="20">
        <a-col :span="12">
          <a-form-item label='商家名称' v-bind="formItemLayout">
            <a-input v-decorator="[
            'name',
            { rules: [{ required: true, message: '请输入商家名称!' }] }
            ]"/>
          </a-form-item>
        </a-col>
        <a-col :span="12">
          <a-form-item label='负责人' v-bind="formItemLayout">
            <a-input v-decorator="[
            'principal',
            { rules: [{ required: true, message: '请输入负责人!' }] }
            ]"/>
          </a-form-item>
        </a-col>
        <a-col :span="12">
          <a-form-item label='店铺地址'>
            <a-input-search
              v-decorator="[
              'address'
              ]"
              enter-button="选择"
              @search="showChildrenDrawer"
            />
          </a-form-item>
        </a-col>
        <a-col :span="12">
          <a-form-item label='经度' v-bind="formItemLayout">
            <a-input v-decorator="[
            'longitude',
            { rules: [{ required: true, message: '请输入经度!' }] }
            ]"/>
          </a-form-item>
        </a-col>
        <a-col :span="12">
          <a-form-item label='纬度' v-bind="formItemLayout">
            <a-input v-decorator="[
            'latitude',
            { rules: [{ required: true, message: '请输入纬度!' }] }
            ]"/>
          </a-form-item>
        </a-col>
        <a-col :span="12">
          <a-form-item label='联系方式' v-bind="formItemLayout">
            <a-input v-decorator="[
            'phone',
            { rules: [{ required: true, message: '请输入联系方式!' }] }
            ]"/>
          </a-form-item>
        </a-col>
        <a-col :span="24">
          <a-form-item label='配送距离阈值（不限制距离请填写-1）/KM' v-bind="formItemLayout">
            <a-input-number style="width: 100%" v-decorator="[
            'distanceThreshold',
            { rules: [{ required: true, message: '请输入配送距离阈值!' }] }
            ]" :min="1" :step="1"/>
          </a-form-item>
        </a-col>
        <a-col :span="24">
          <a-form-item label='营业星期' v-bind="formItemLayout">
            <div :style="{ borderBottom: '1px solid #E9E9E9' }">
              <a-checkbox :indeterminate="indeterminate" :checked="checkAll" @change="onCheckAllChange">
                Check all
              </a-checkbox>
            </div>
            <br />
            <a-checkbox-group v-model="checkedList" :options="plainOptions" @change="onChange" />
          </a-form-item>
        </a-col>
        <a-col :span="12">
          <a-form-item label='开始营业时间' v-bind="formItemLayout">
            <a-time-picker :default-open-value="moment('00:00:00', 'HH:mm:ss')" style="width: 100%" v-decorator="[
            'operateStartTime',
            { rules: [{ required: true, message: '请输入开始营业时间!' }] }
            ]" />
          </a-form-item>
        </a-col>
        <a-col :span="12">
          <a-form-item label='营业结束时间' v-bind="formItemLayout">
            <a-time-picker style="width: 100%" v-decorator="[
            'operateEndTime',
            { rules: [{ required: true, message: '请输入营业结束时间!' }] }
            ]" />
          </a-form-item>
        </a-col>
        <a-col :span="24">
          <a-form-item label='店铺介绍' v-bind="formItemLayout">
            <a-textarea :rows="6" v-decorator="[
            'content',
             { rules: [{ required: true, message: '请输入店铺介绍!' }] }
            ]"/>
          </a-form-item>
        </a-col>
        <a-col :span="24">
          <a-form-item label='图册' v-bind="formItemLayout">
            <a-upload
              name="avatar"
              action="http://127.0.0.1:9527/file/fileUpload/"
              list-type="picture-card"
              :file-list="fileList"
              @preview="handlePreview"
              @change="picHandleChange"
            >
              <div v-if="fileList.length < 8">
                <a-icon type="plus" />
                <div class="ant-upload-text">
                  Upload
                </div>
              </div>
            </a-upload>
            <a-modal :visible="previewVisible" :footer="null" @cancel="handleCancel">
              <img alt="example" style="width: 100%" :src="previewImage" />
            </a-modal>
          </a-form-item>
        </a-col>
      </a-row>
    </a-form>
    <drawerMap :childrenDrawerShow="childrenDrawer" @handlerClosed="handlerClosed"></drawerMap>
    <div class="drawer-bootom-button">
      <a-popconfirm title="确定放弃编辑？" @confirm="onClose" okText="确定" cancelText="取消">
        <a-button style="margin-right: .8rem">取消</a-button>
      </a-popconfirm>
      <a-button key="submit" type="primary" :loading="loading" @click="handleSubmit">
        提交
      </a-button>
    </div>
  </a-drawer>
</template>

<script>
import baiduMap from '@/utils/map/baiduMap'
import drawerMap from '@/utils/map/searchmap/drawerMap'
import {mapState} from 'vuex'
import moment from 'moment'
moment.locale('zh-cn')
function getBase64 (file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.readAsDataURL(file)
    reader.onload = () => resolve(reader.result)
    reader.onerror = error => reject(error)
  })
}
const plainOptions = ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
const formItemLayout = {
  labelCol: { span: 24 },
  wrapperCol: { span: 24 }
}
export default {
  name: 'merchantEdit',
  props: {
    merchantEditVisiable: {
      default: false
    }
  },
  components: {
    drawerMap
  },
  computed: {
    ...mapState({
      currentUser: state => state.account.user
    }),
    show: {
      get: function () {
        return this.merchantEditVisiable
      },
      set: function () {
      }
    }
  },
  data () {
    return {
      checkedList: [],
      indeterminate: true,
      checkAll: false,
      plainOptions,
      rowId: null,
      formItemLayout,
      form: this.$form.createForm(this),
      loading: false,
      fileList: [],
      previewVisible: false,
      previewImage: '',
      localPoint: {},
      staymerchant: '',
      childrenDrawer: false
    }
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
    handlerClosed (localPoint) {
      this.childrenDrawer = false
      if (localPoint !== null && localPoint !== undefined) {
        this.localPoint = localPoint
        console.log(this.localPoint)

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
            let obj = {}
            obj['address'] = this.stayAddress
            obj['longitude'] = localPoint.lng
            obj['latitude'] = localPoint.lat
            this.form.setFieldsValue(obj)
          }
        })
      }
    },
    showChildrenDrawer (value) {
      this.flagType = value
      this.childrenDrawer = true
    },
    onChildrenDrawerClos () {
      this.childrenDrawer = false
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
    imagesInit (images) {
      if (images !== null && images !== '') {
        let imageList = []
        images.split(',').forEach((image, index) => {
          imageList.push({uid: index, name: image, status: 'done', url: 'http://127.0.0.1:9527/imagesWeb/' + image})
        })
        this.fileList = imageList
      }
    },
    setFormValues ({...merchant}) {
      this.rowId = merchant.id
      let fields = ['name', 'dishes', 'code', 'operateEndTime', 'operateStartTime', 'principal', 'longitude', 'latitude', 'address', 'phone', 'content', 'operateDay', 'distanceThreshold']
      let obj = {}
      Object.keys(merchant).forEach((key) => {
        if (key === 'images') {
          this.fileList = []
          this.imagesInit(merchant['images'])
        }
        if (key === 'operateDay' && merchant[key] != null) {
          let operateDay = merchant[key].split(',')
          let checkList = []
          operateDay.forEach(e => {
            switch (e) {
              case '1':
                checkList.push('周一')
                break
              case '2':
                checkList.push('周二')
                break
              case '3':
                checkList.push('周三')
                break
              case '4':
                checkList.push('周四')
                break
              case '5':
                checkList.push('周五')
                break
              case '6':
                checkList.push('周六')
                break
              case '7':
                checkList.push('周日')
                break
            }
          })
          this.checkedList = checkList
        }
        if (key === 'operateStartTime' && merchant[key] != null) {
          merchant[key] = moment(merchant[key], 'HH:mm:ss')
        }
        if (key === 'operateEndTime' && merchant[key] != null) {
          merchant[key] = moment(merchant[key], 'HH:mm:ss')
        }
        if (fields.indexOf(key) !== -1) {
          this.form.getFieldDecorator(key)
          obj[key] = merchant[key]
        }
      })
      this.form.setFieldsValue(obj)
    },
    reset () {
      this.loading = false
      this.form.resetFields()
    },
    onClose () {
      this.reset()
      this.$emit('close')
    },
    handleSubmit () {
      if (this.checkedList.length === 0) {
        this.$message.warn('至少选择一天')
        return false
      }
      // 获取图片List
      let images = []
      this.fileList.forEach(image => {
        if (image.response !== undefined) {
          images.push(image.response)
        } else {
          images.push(image.name)
        }
      })
      this.form.validateFields((err, values) => {
        values.id = this.rowId
        values.images = images.length > 0 ? images.join(',') : null
        values.operateDay = this.checkedList.join(',')
        values.operateStartTime = moment(values.operateStartTime).format('HH:mm:ss')
        values.operateEndTime = moment(values.operateEndTime).format('HH:mm:ss')
        if (!err) {
          this.loading = true
          this.$put('/cos/merchant-info', {
            ...values
          }).then((r) => {
            this.reset()
            this.$emit('success')
          }).catch(() => {
            this.loading = false
          })
        }
      })
    }
  }
}
</script>

<style scoped>

</style>
