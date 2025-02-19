<template>
  <view class="page-index">
    <!-- 搜索框 -->
    <view class="search-box">
      <view
        class="search"
        @tap="toSrearch"
      >
        <view class="icon">
          <image src="/static/images/search.png" />
        </view>
        <view class="text">
          搜索您想购买的商品或者店铺
        </view>
      </view>
    </view>

    <!-- banner -->
    <view class="banner">
      <view class="bg">
        <image src="/static/images/bannerBg.png" />
      </view>
      <swiper
        class="img"
        indicator-dots="true"
        indicator-color="rgba(255,252,255, .3)"
        indicator-active-color="rgba(255,252,255, .6)"
        autoplay="true"
        circular="true"
      >
        <swiper-item
          v-for="(imgItem, imgIdx) in indexImgs"
          :key="imgIdx"
          class="item"
          @tap="toProdDetail(imgItem.spuId)"
        >
          <image :src="imgItem.imgUrl" />
        </swiper-item>
      </swiper>
    </view>

    <!-- 导航 -->
    <view class="nav-box">
      <view
        class="item"
        @tap="toNewProds"
      >
        <view class="icon">
          <image src="/static/images/icon-new.png" />
        </view>
        <view class="text">
          新品推荐
        </view>
      </view>
      <view
        class="item"
        @tap="notOpen('打折特惠')"
      >
        <view class="icon">
          <image src="/static/images/icon-sale.png" />
        </view>
        <view class="text">
          打折特惠
        </view>
      </view>
      <view
        class="item"
        @tap="notOpen('优惠团购')"
      >
        <view class="icon">
          <image src="/static/images/icon-group.png" />
        </view>
        <view class="text">
          优惠团购
        </view>
      </view>
      <view
        class="item"
        @tap="notOpen('秒杀')"
      >
        <view class="icon">
          <image src="/static/images/icon-flash.png" />
        </view>
        <view class="text">
          限时秒杀
        </view>
      </view>
      <view
        class="item"
        @tap="notOpen('领券中心')"
      >
        <view class="icon">
          <image src="/static/images/icon-coupon.png" />
        </view>
        <view class="text">
          领券中心
        </view>
      </view>
      <view
        class="item"
        @tap="notOpen('会员中心')"
      >
        <view class="icon">
          <image src="/static/images/icon-member.png" />
        </view>
        <view class="text">
          会员中心
        </view>
      </view>
      <view
        class="item"
        @tap="notOpen('分销中心')"
      >
        <view class="icon">
          <image src="/static/images/icon-distribution.png" />
        </view>
        <view class="text">
          分销中心
        </view>
      </view>
      <view
        class="item"
        @tap="notOpen('积分商城')"
      >
        <view class="icon">
          <image src="/static/images/icon-integral.png" />
        </view>
        <view class="text">
          积分商城
        </view>
      </view>
    </view>

    <!-- 推荐 -->
    <view class="recommend">
      <scroll-view
        scroll-x="true"
        class="category"
        :scroll-left="categoryScrollLeft"
        scroll-with-animation
        @scroll="scroll"
      >
        <block
          v-for="(item, index) in categoryList"
          :key="index"
        >
          <view
            class="category-item"
            :class="{active:selectedIndex===index}"
            :data-index="index"
            :data-categoryid="item.categoryId"
            @tap="switchCategory"
          >
            {{ item.name }}
          </view>
        </block>
      </scroll-view>
    </view>
    <view class="prods">
      <block
        v-for="(item, index) in categoryProdList"
        :key="index"
      >
        <view
          class="item"
          @tap="toProdDetail(item.spuId)"
        >
          <view class="img">
            <image :src="item.mainImgUrl" />
          </view>
          <view class="text-box">
            <view class="name">
              {{ item.spuName }}
            </view>
            <view class="price-box">
              <view class="price">
                <view class="symbol">
                  ￥
                </view>
                <view class="big">
                  {{ wxs.parsePrice(item.priceFee)[0] }}
                </view>
                <view class="symbol">
                  .{{ wxs.parsePrice(item.priceFee)[1] }}
                </view>
              </view>
            </view>
          </view>
        </view>
      </block>
    </view>
    <!-- 无商品显示 -->
    <view
      v-if="categoryProdList.length===0"
      class="empty"
    >
      <view class="img">
        <image src="/static/empty-img/not-found.png" />
      </view>
      <view class="text">
        没有找到对应商品，看看别的吧
      </view>
    </view>

    <view
      v-if="isLoadAll && categoryProdList.length>0"
      class="nomore"
    >
      没有更多了，看看别的吧
    </view>
  </view>
</template>

<script setup>
import { reactive } from 'vue'
import Wechat from '@/utils/wechat.js'
import * as cartCount from '@/utils/cart-count.js'
const wxs = number()

const Data = reactive({
  picDomain: import.meta.env.VITE_APP_RESOURCES_URL, // 图片前缀
  // 查询的参数
  pageQuery: {
    shopId: 0
  },
  // 返回参数
  noticeData: {
    list: [], // 返回的列表
    total: 0, // 一共多少条数据
    pages: 0 // 一共多少页
  },
  indexImgs: [], // 轮播图
  categoryList: [], // 分类列表
  selectedIndex: 0, // 选中的分类项(默认为第一个分类)
  selectedCategoryId: '',
  // 分类商品列表的请求参数
  searchListQuery: {
    pageSize: 10,
    pageNum: 1,
    primaryCategoryId: '',
    sort: 1 // 新品 1:新品
  },
  // 分类商品列表返回的参数
  searchListData: {
    list: [],
    total: 0, // 总数
    pages: 0 // 总页数
  },
  categoryProdList: [], // 分类的商品列表
  isLoadAll: false, // 是否加载全部
  // 分类滚动栏横向滚动距离
  categoryScrollLeft: 0,
  // 分类滚动的总宽度
  contentScrollW: 0,
  isWechat: false // 是否为微信环境
})
const { indexImgs, categoryList, selectedIndex, categoryProdList, isLoadAll, categoryScrollLeft } = toRefs(Data)

onShow(() => {
  const tempuid = uni.getStorageSync('cloudTempUid')
  const token = uni.getStorageSync('cloudToken')

  // #ifdef MP-WEIXIN
  if (!(!tempuid && !token)) {
    queryDataGroup()
  }
  // #endif

  // #ifdef H5
  Data.isWechat = Wechat.isWechat()
  if (Data.isWechat) { // H5-微信环境
    if (!tempuid && !token) {
      http.mpAuthLogin('/', true)
      // alert('公众号网页登录location.href:' + location.href)
      const code = util.getUrlKey('code')
      // alert('H5-微信环境 code:' + code)
      login('/mall4cloud_auth/ua/social/mp', code) // 先请求微信公众号登录接口
    } else {
      queryDataGroup()
    }
  } else {
    queryDataGroup()
  }
  // #endif

  // 获取购物车数字
  cartCount.getCartCount()
})

onLoad(() => {
  queryDataGroup()
})

/**
 * 页面上拉触底事件的处理函数
 */
onReachBottom(() => {
  if (Data.searchListQuery.pageNum < Data.searchListData.pages) {
    Data.searchListQuery.pageNum = Data.searchListQuery.pageNum + 1
    getSearchList()
  } else {
    Data.isLoadAll = true
  }
})

/**
     * 跳转商品详情
     */
const toProdDetail = (spuId) => {
  if (spuId) {
    uni.navigateTo({
      url: '/pages/detail/detail?spuId=' + spuId
    })
  }
}

/**
     * 微信小程序/公众号登录
     */
const login = (url, code) => {
  // 发送 code 到后台换取 token
  http.request({
    login: true,
    url,
    data: code
  }).then((res) => {
    uni.setStorageSync('cloudToken', res.accessToken)
    uni.setStorageSync('cloudLoginResult', res) // 保存整个登录数据
    queryDataGroup() // 查询页面所有数据
  }).catch(err => {
    // 异常
    uni.hideLoading()
    if (err.code === 'A04001') {
      uni.setStorageSync('cloudTempUid', err.data) // token或tempUid
      // 社交账号未绑定
      uni.showModal({
        title: '提示',
        content: '还未绑定账号，是否前往绑定？',
        success: data => {
          if (data.confirm) { // 确认
            uni.navigateTo({
              url: '/pages/bind-account/bind-account'
            })
          }
        }
      })
    }
  })
}

/**
     * 查询页面数据方法合集
     */
const queryDataGroup = () => {
  getIndexImgs()
  getPlatformCategoryList()
}

/**
     * 轮播图
     */
const getIndexImgs = () => {
  uni.showLoading()
  http.request({
    url: '/mall4cloud_multishop/ua/index_img/list',
    method: 'GET',
    data: {
      shopId: Data.pageQuery.shopId
    }
  }).then((res) => {
    uni.hideLoading()
    Data.indexImgs = res
  })
}

/**
     * 获取商品列表
     */
const getSearchList = () => {
  uni.showLoading()
  Data.isLoadAll = false
  Data.searchListQuery.primaryCategoryId = Data.selectedCategoryId
  http.request({
    url: '/mall4cloud_search/ua/search/simple_page',
    method: 'GET',
    data: Data.searchListQuery
  }).then(res => {
    Data.searchListData = res
    let list = []
    if (Data.searchListQuery.pageNum === 1) {
      list = res.list[0].spus
    } else {
      list = Data.categoryProdList
      list = list.concat(res.list[0].spus)
    }
    if (Data.searchListQuery.pageNum === Data.searchListData.pages) {
      Data.isLoadAll = true
    }
    Data.categoryProdList = list
    uni.hideLoading()
  })
}

/**
     * 获取分类栏 scroll-view 滚动相关宽度
     */
const getCategoryScrollWidth = () => {
  const query = uni.createSelectorQuery()
  query.select('.category').boundingClientRect(data => {
    // scroll-view 的宽度
    Data.contentScrollW = data.width
  }).exec()
  // 后面优化下这个地方
  setTimeout(() => {
    query.selectAll('.category-item').boundingClientRect(data => {
      const dataLen = data.length
      for (let i = 0; i < dataLen; i++) {
        // scroll-view 各个分类项目的距离左边栏的距离
        Data.categoryList[i].left = data[i].left
        // scroll-view 各个分类项目的宽度(看样式那里原来写死了20%，我改成自动宽度)
        Data.categoryList[i].width = data[i].width
      }
    }).exec()
  }, 1)
}

/**
     * 获取平台分类
     */
const getPlatformCategoryList = () => {
  uni.showLoading()
  http.request({
    url: '/mall4cloud_product/ua/category/category_list',
    method: 'GET',
    data: {
      shopId: 0,
      parentId: 0
    }
  }).then(res => {
    Data.categoryList = res
    getSearchList()
    uni.hideLoading()
    getCategoryScrollWidth()
  })
}

/**
     * 切换分类
     */
const switchCategory = (e) => {
  const index = parseInt(e.currentTarget.dataset.index)
  Data.selectedCategoryId = parseInt(e.currentTarget.dataset.categoryid)
  if (Data.selectedIndex === index) {
    return
  }
  Data.selectedIndex = index
  Data.searchListQuery.pageNum = 1
  // scorll-view 实现居中显示的滚动位置(偏移值): 当前点击子元素距离左边栏的距离 - scroll-view 宽度的一半  + 当前点击子元素一半的宽度 - 父盒子边距(如有)
  Data.categoryScrollLeft = Data.categoryList[index].left - Data.contentScrollW / 2 + Data.categoryList[index].width / 2 - 15
  getSearchList()
}

const scroll = (e) => {
  setTimeout(() => {
    Data.categoryScrollLeft = e.target.scrollLeft
  })
}

// 跳转搜索页
const toSrearch = () => {
  uni.navigateTo({
    url: '/pages/search-page/search-page'
  })
}

// 跳转新品推荐
const toNewProds = () => {
  uni.navigateTo({
    url: '/pages/new-prods/new-prods'
  })
}

const notOpen = (title) => {
  uni.showModal({
    title: '提示',
    content: `${title}暂未开源`,
    showCancel: false
  })
}

</script>

<style lang="scss" scoped>
@use "index";
</style>
