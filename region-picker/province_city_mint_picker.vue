<style lang="less">
  @import '../assets/less/picker';
</style>

<template>
  <scroller @confirm="cityConfirm" ref="cityScroller" @change="changeVal"  :scroll="scrollerCityData"></scroller>
</template>

<script type="text/javascript">
  import CityData from './citydata.json'
  import Scroller from 'anima-vue-scroller'

  CityData[0].code !== '0' && CityData.unshift({
    code: '0',
    name: '全国',
    children: [
      {
        code: '0',
        name: '不限'
      }
    ]
  })
  const processCityData = {
    setUnlimitedData (data) {
      var unlimited = {
        code: 0,
        name: '不限'
      }
      if (Array.isArray(data)) {
        data.forEach((item) => {
          if (item.children.length > 1 && item.children[0].code !== '0') {
            item.children.unshift(unlimited)
          }
        })
      }
      return data
    }
  }
  processCityData.setUnlimitedData(CityData)

  export default {
    components: {
      Scroller
    },
    props: {
      province: {
        default: '0'
      },
      city: {
        default: '0'
      },
      region: {
        default: '0'
      },
      /**
       * 点击确定
       */
      onOk: {
        type: Function,
        required: false,
        default: undefined
      },
      /**
       * 点击取消
       */
      onCancel: {
        type: Function,
        required: false,
        default: undefined
      }
    },
    data () {
      return {
        scrollerCityData: [{
          defaultValue: '',
          data: []
        }, {
          defaultValue: '',
          data: []
        }, {
          defaultValue: '',
          data: []
        }],
        selectAddressValue: []
      }
    },
    created () {
      this.getProvince(this.province)
      this.getCity(this.scrollerCityData[0].defaultValue, this.city)
      this.getRegion(this.scrollerCityData[0].defaultValue, this.scrollerCityData[1].defaultValue, this.region)
    },
    methods: {
      changeVal (values) {
        const that = this
        let defaultValue = that.selectAddressValue[0] ? that.selectAddressValue[0].value : ''
        let defaultValues = that.selectAddressValue[1] ? that.selectAddressValue[1].value : ''
        if (values[0].value !== defaultValue) {
          that.selectAddressValue = []
          values.forEach((item) => {
            that.selectAddressValue.push(item)
          })
          that.getCity(values[0].value)
        }
        if (values[1].value !== defaultValues) {
          that.selectAddressValue = []
          values.forEach((item) => {
            that.selectAddressValue.push(item)
          })
          that.getRegion(values[0].value, values[1].value)
        }
      },
      cityConfirm (values) {
        this.$emit('on-ok', values)
        this.selectAddressValue = []
        values.forEach((item) => {
          this.selectAddressValue.push(item)
        })
      },
      getProvince (value) {
        const that = this
        var arr = []
        CityData.forEach((item) => {
          arr.push({
            value: item.code,
            name: item.name
          })
        })
        that.scrollerCityData[0].defaultValue = value || arr[0].value
        that.scrollerCityData[0].data = arr
      },
      getCity (data, value) {
        const that = this
        var arr = []
        CityData.forEach((item) => {
          if (data === item.code) {
            item.children.forEach(key => {
              arr.push({
                value: key.code,
                name: key.name
              })
            })
          }
        })
        that.scrollerCityData[1].defaultValue = value || arr[0].value
        that.scrollerCityData[1].data = arr
      },
      getRegion (provinceData, cityData, value) {
        const that = this
        var arr = []
        CityData.forEach((item) => {
          if (provinceData === item.code) {
            if (item.children.length === 1) {
              item.children.forEach(key => {
                key.children.forEach(keys => {
                  arr.push({
                    value: keys.code,
                    name: keys.name
                  })
                })
              })
            } else {
              item.children.forEach(key => {
                if (key.code === 0) {
                  arr.push({
                    value: 0,
                    name: '不限'
                  })
                } else if (cityData === key.code) {
                  key.children.forEach(keys => {
                    arr.push({
                      value: keys.code,
                      name: keys.name
                    })
                  })
                }
              })
            }
          }
        })
        that.scrollerCityData[2].defaultValue = value || arr[0].value
        that.scrollerCityData[2].data = arr
      },
      /**
       * 这是一个对外公开的方法，用于显示选择器。
       */
      show () {
        this.$refs['cityScroller'].open()
      }
    }
  }
</script>
