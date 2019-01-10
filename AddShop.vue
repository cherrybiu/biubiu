<template>
  <div class="page h-100 bg-main pdd-tb-10">
    <div class="shadow" v-if='showShadow'></div>
    <div class="main bg-fff pdd-lr-10" :class="{'hasBottom':!showBn}" @scroll="scrollAction($event)">
      <div class="menu-group fs-14">
        <div class="menu flex lh-50 cm-border cm-border-bottom">
          <div class="f-ft c-999">门店名称</div>
          <input type="text" placeholder="请输入品牌商名称" class="f-content c-45 fs-14" v-model="shop.name">
        </div>
        <div class="menu flex lh-50 cm-border cm-border-bottom">
          <div class="f-ft c-999">门店电话</div>
          <input type="text" placeholder="请输入联系电话" class="f-content c-45 fs-14" @keyup="formatPhone(shop.phone)" v-model="shop.phone" maxlength='12' min-length='11'>
        </div>
        <div class="menu flex lh-50 cm-select cm-border cm-border-bottom  fs-14">
          <div class="f-ft c-999">营业时间</div>
          <div class="f-content">
            <span class="time" @click="selectStart">{{startTime||'开始时间'}}</span><span>至</span>
            <span @click="showSelect">{{openDate}}</span>
            <span class="time" @click="selectEnd">{{endTime||'结束时间'}}</span>
          </div>
          <!-- <mt-picker :slots="slots" @change="onValuesChange" v-if='showDate'></mt-picker> -->
        </div>
        <div class="menu flex lh-50 cm-border cm-border-bottom cm-select">
          <div class="f-ft c-999">经营类目</div>
          <div class="f-content over-ellipse" @click="emitClassify" v-text="formatTypeName()" :class="{'selected':!oneType}">请选择</div>
        </div>
        <div class="menu textarea flex lh-50 cm-select cm-border cm-border-bottom  fs-14">
          <div class="f-ft c-999">门店地址</div>
          <div class="f-content over-ellipse" @click="selectBankLoc" v-text="bankLoc?bankLoc:'请选择'" :class="{'selected':!bankLoc}">请选择</div>
        </div>
        <div class="menu  textareaMenu flex lh-50 cm-border cm-border-bottom  fs-14">
          <div class="f-ft c-999">详细地址</div>
          <textarea cols="30" rows="3" placeholder="街道、楼牌号等" class="f-content fs-14" v-model="shop.detailAddress"></textarea>
        </div>

        <!-- <div class="logo-menu">
          <div class="c-999 lh-50">门店门头照</div>
          <div class="wrap-img flex cm-border-top">
            <div class="left-img f-content">
              <comp-camera cameraName='avatar' :cameraNums="logoArr" :isCrop="isCrop" @translatePermit='transImg'></comp-camera>
            </div>
            <div class="tip f-content fs-12">
              注：<br>门头照图片尺寸比例1:1，不超过5M; 支持png/jpeg/jpg/ bmp格式
            </div>
          </div>
        </div> -->

        <div class="menu-group bg-fff pdd-10">
          <h3 class="lh-40 cm-border cm-border-bottom">门头照</h3>
          <div class="wrap-slide">
            <auth-camera :cameraNums="shopImgArr" cameraName="shop" @translatePermit="getShopImg"></auth-camera>
          </div>
        </div>

        <div class="menu bg-fff pdd-10">
          <h3 class="lh-40 cm-border cm-border-bottom">营业执照</h3>
          <div class="wrap-slide">
            <auth-camera :cameraNums="licenseImg" cameraName="license" @translatePermit="getPermit"></auth-camera>
          </div>
        </div>
      </div>
    </div>

    <div class="footer bg-fff">
      <div class="bn c-fff bg-blue t-center lh-40" @click="addShop">保存</div>
    </div>

    <!-- 时间选择框 -->
    <mt-popup v-model="showDate" position="bottom" class="mint-popup">
      <mt-picker :slots="slots" :visible-item-count="5" :show-toolbar="true" ref="picker">
        <mt-button @click="handleDateConfirm" class="sure">确认</mt-button>
      </mt-picker>
    </mt-popup>
    <mt-datetime-picker ref="pickerStart" type="time" v-model="startTime" @confirm="handleConfirm"></mt-datetime-picker>
    <mt-datetime-picker ref="pickerEnd" type="time" v-model="endTime" @confirm="handleConfirmEnd"></mt-datetime-picker>

    <!-- 地点一 -->
    <div class="classify-list cm-border cm-border-top pdd-lr-10" :class="{slideUp:showBankLoc,slideDown:closeBankLoc}">
      <linkage @close='closeBankDialog' selectName='second' @transVal="assignBankLoc"></linkage>
    </div>

    <!-- 选择分类 -->
    <div class="classify-list pdd-lr-10 cm-border cm-border-top" :class="{slideUp:showClassify,slideDown:closeClassify}">
      <div class="pdd-10 flex f-end">
        <div class="close c-999 fs-12" @click="close">关闭</div>
      </div>
      <div class="selectList cm-border cm-border-bottom flex fs-12 lh-20">
        <div class="loc" v-if="oneType" @click="showOne" :class="{active:parseInt(this.typeIndex) === 1}">{{oneType}}</div>
        <div class="loc" v-if="twoType" @click="showTwo" :class="{active:parseInt(this.typeIndex) === 2}">{{twoType}}</div>
        <div class="loc" v-if="threeType" @click="showThree" :class="{active:parseInt(this.typeIndex) === 3}">{{threeType}}</div>
        <div class="loc c-blue" v-if="!threeType" :class="{active:parseInt(this.typeIndex) === 0}">请选择</div>
      </div>
      <div class="list-group flex" :class="{slideTwo:activeIndex === 2,slideThree:activeIndex === 3}">
        <div class="list">
          <div class="cell lh-30 fs-12" @click="selectOneType(province,i)" v-for="(province,i) in oneList" :key="i" :class="{'c-blue':oneIndex == i}">{{province.name}}</div>
        </div>
        <div class="list">
          <div class="cell lh-30 fs-12" @click="selectTwoType(city,i)" v-for="(city,i) in twoList" :key="i" :class="{'c-blue':twoIndex == i}">{{city.name}}</div>
        </div>
        <div class="list">
          <div class="cell lh-30 fs-12" @click="selectThreeType(town,i)" v-for="(town,i) in threeList" :key="i" :class="{'c-blue':threeIndex == i}">{{town.name}}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import AuthCamera from '../common/AuthCamera';
import Linkage from '../common/Linkage';
import { Toast } from 'mint-ui';
// import moment from 'moment'// 格式化时间
export default {
  components: { Linkage, AuthCamera },
  async mounted() {
    document.body.addEventListener('focusout', () => {
      // 软键盘收起的事件处理
      if (this.$util.isIphone()) {
        this.isPopup = false;
        this.scrollToEvent();
      }
    });
    document.body.addEventListener('focusin', () => {
      if (this.$util.isIphone()) {
        this.isPopup = true;
      }
    });
    await this.getCategory(0);
  },
  data() {
    return {
      shop: {
        name: '',
        logo_img: '',
        phone: '',
        cate_id_1: '',
        cate_id_2: '',
        classify: '',
        detailAddress: '',
        permitNum: '',
        licenImg: ''
      },
      provinceName: '',
      logoArr: [{ name: 'image1' }],
      isCrop: true,
      bankLoc: '',
      showLoc: '',
      closeLoc: '',
      showShadow: false,
      closeBankLoc: false,
      showBankLoc: false,
      //   类别
      showClassify: false,
      closeClassify: false,
      oneType: '',
      twoType: '',
      threeType: '',
      typeIndex: null,
      oneList: [{ name: '111', id: 1 }, { name: '111', id: 1 }],
      twoList: [{ name: '222', id: 1 }, { name: '222', id: 1 }],
      threeList: [{ name: '333', id: 1 }, { name: '333', id: 1 }],
      oneIndex: null,
      twoIndex: null,
      threeIndex: null,
      type: '',
      activeIndex: null,
      startTime: '',
      endTime: '',
      slots: [{ values: ['今日', '次日'] }],
      showDate: false,
      openDate: '今日',
      area_code: [],
      oneId: '',
      twoId: '',
      threeId: '',
      isPopup: false,
      showBn: true,
      licenseImg: [{ name: 'license', txt: '营业执照图片' }],
      shopImgArr: [{ name: 'shop', txt: '商户门头照' }]
    };
  },
  methods: {
    isHide(val) {
      this.showBn = val;
    },
    // IOS系统下微信平台收起键盘后控制网页复原位置的事件
    scrollToEvent() {
      let self = this;
      let timer = setTimeout(() => {
        if (!self.isPopup) {
          window.scrollTo({ left: 0, top: self.offsetTop, behavior: 'smooth' });
        }
        clearTimeout(timer);
      }, 0);
    },
    scrollAction(e) {
      this.offsetTop = e.target.scrollTop;
    },
    // 选择时间
    selectStart() {
      this.$refs.pickerStart.open();
    },
    selectEnd() {
      this.$refs.pickerEnd.open();
    },
    handleConfirm(date) {
      // let date = moment(data).format('YYYY.MM.DD')
      this.startTime = date;
    },
    handleConfirmEnd(date) {
      // let date = moment(data).format('YYYY.MM.DD')
      this.endTime = date;
    },
    handleDateConfirm() {
      this.openDate = this.$refs.picker.getValues()[0];
      this.showDate = false;
    },
    showSelect() {
      this.showDate = true;
    },
    // 规范手机号
    formatPhone(phone) {
      this.shopPhone = phone.replace(/\D/g, '');
    },
    // 获取分类
    async getCategory(val) {
      this.oneList = await this.$http.post('/user/category', { pid: val });
    },
    selectBankLoc() {
      this.showBankLoc = true;
      this.closeBankLoc = false;
      this.showShadow = true;
    },
    closeBankDialog() {
      this.showBankLoc = false;
      this.closeBankLoc = true;
      this.showShadow = false;
    },
    assignBankLoc(val, arr) {
      this.bankLoc = val;
      this.area_code = arr;
    },
    // 触发选择类
    emitClassify() {
      this.showClassify = true;
      this.closeClassify = false;
      this.showShadow = true;
    },
    close() {
      this.showClassify = false;
      this.closeClassify = true;
      this.showShadow = false;
    },
    // 类别选择
    showOne() {
      this.typeIndex = 1;
      this.activeIndex = 0;
    },
    showTwo() {
      this.typeIndex = 2;
      this.activeIndex = 2;
    },
    showThree() {
      this.typeIndex = 3;
      this.activeIndex = 3;
    },
    async selectOneType(type, i) {
      this.twoList = await this.$http.post('/user/category', {
        pid: type.id
      });
      if (type.name !== this.oneType) {
        this.twoType = '';
        this.threeType = '';
        this.twoIndex = null;
        this.threeIndex = null;
      }
      this.oneIndex = i;
      if (!this.twoType) {
        this.typeIndex = 0;
      } else {
        this.typeIndex = 2;
      }
      this.activeIndex = 2;
      this.oneType = type.name;
      this.oneId = type.id;
    },
    async selectTwoType(type, i) {
      this.threeList = await this.$http.post('/user/category', {
        pid: type.id
      });
      if (type.name !== this.twoType) {
        this.threeType = '';
        this.threeIndex = null;
      }
      this.twoIndex = i;
      if (!this.threeType) {
        this.typeIndex = 0;
      } else {
        this.typeIndex = 3;
      }
      this.activeIndex = 3;
      this.twoType = type.name;
      this.twoId = type.id;
    },
    selectThreeType(type, i) {
      this.threeIndex = i;
      this.threeType = type.name;
      this.threeId = type.id;
      this.typeIndex = 3;
      this.showShadow = false;
      this.closeClassify = true;
      this.showClassify = false;
    },
    formatTypeName() {
      let twoType = '';
      let threeType = '';
      if (this.twoType) {
        twoType = '-' + this.twoType;
      }
      if (this.threeType) {
        threeType = '-' + this.threeType;
      }
      this.type = this.oneType + twoType + threeType;
      return this.oneType ? this.oneType + twoType + threeType : '请选择';
    },
    // 图片赋值
    // transImg(val) {
    //   this.shop.logo_img = val;
    // },
    getShopImg(data) {
      this.shop.logo_img = data;
    },
    //组件传值
    getPermit(data) {
      this.shop.licenImg = data;
    },
    async addShop() {
      if (!/^[014]\d{9,11}$/.test(this.shop.phone)) {
        return Toast('电话号码格式不正确');
      }
      let params = {
        shop_name: this.shop.name,
        provinces:
          this.bankLoc === '请选择' ? '' : this.bankLoc.replace(/-/g, ''),
        area_code: this.area_code.join(','),
        address: this.shop.detailAddress,
        service_phone: this.shop.phone,
        shop_time: this.startTime + '至' + this.openDate + this.endTime,
        business_category: this.threeId,
        logo_img: this.shop.logo_img
      };
      await this.$http.post('/user/addshop', params);
      Toast('添加门店成功');
      this.$router.go(-1);
    }
  }
};
</script>

<style lang="less" scoped>
.page {
  overflow: hidden;
  box-sizing: border-box;
  position: relative;

  .v-modal {
    bottom: 0;
  }

  .main {
    position: absolute;
    left: 0;
    right: 0;
    top: 10px;
    bottom: 60px;
    z-index: 1;
    overflow-y: scroll;

    &.hasBottom {
      bottom: 0;
    }
  }

  .shadow {
    width: 100%;
    height: 100%;
    position: absolute;
    background: rgba(0, 0, 0, 0.3);
    z-index: 10;
    top: 0;
    left: 0;
  }

  .classify-list {
    width: 100%;
    height: 264px;
    background: #fff;
    position: absolute;
    bottom: -264px;
    left: 0;
    z-index: 100;
    box-sizing: border-box;

    .selectList {
      padding-bottom: 5px;
      overflow-y: scroll;
      .loc {
        margin-right: 30px;
        position: relative;
        &.active:after {
          position: absolute;
          content: '';
          bottom: -4px;
          left: 0;
          width: 100%;
          height: 1px;
          background: #21b5ff;
        }
      }
    }
    .list-group {
      width: 310%;
      transition: all 0.5s;
      padding: 10px 0;

      &.slideTwo {
        transform: translate(-33.3%, 0);
      }
      &.slideThree {
        transform: translate(-66.6%, 0);
      }

      .list {
        width: 50%;
        padding: 0 20px 0 0;
        height: 180px;
        overflow-y: scroll;
      }
    }

    @keyframes slideUp {
      from {
        bottom: -264px;
      }
      to {
        bottom: 0;
      }
    }
    @keyframes slideDown {
      from {
        bottom: 0;
      }
      to {
        bottom: -264px;
      }
    }
    &.slideUp {
      animation: slideUp 0.5s forwards;
      -webkit-animation: slideUp 0.5s forwards;
    }
    &.slideDown {
      animation: slideDown 0.5s forwards;
      -webkit-animation: slideDown 0.5s forwards;
    }
    .list {
      height: 160px;
      overflow-y: scroll;
    }
  }

  .menu {
    .f-ft {
      width: 100px;
    }
    .time {
      display: inline-block;
      padding: 0 10px;
      text-align: center;
    }
    &.textareaMenu {
      height: 90px;
      textarea {
        font-family: PingFang-SC-Medium;
        padding-top: 12px;
        border: none;
        outline: none;
        line-height: 23px;
      }
    }
    .idInput {
      font-size: 14px;
      width: 100px;
    }

    @media (max-width: 320px) {
      .idInput {
        width: 80px;
      }
      .f-ft {
        width: 65px;
      }
    }
    .checkMessage {
      color: rgba(255, 182, 41, 1);
      float: right;
    }
    .selected {
      color: #ccc;
    }
  }
  .logo-menu {
    .tip {
      padding: 15px 0px;
      color: rgba(174, 174, 174, 1);
      line-height: 18px;
      font-size: 12px;
      width: 161px;
    }
    .wrap-img {
      padding-bottom: 10px;
    }
  }
  .footer {
    position: fixed;
    bottom: 0px;
    left: 0px;
    width: 100%;
    box-sizing: border-box;
    padding: 10px 20px;
    z-index: 0;

    .bn {
      border-radius: 6px;
    }
  }
}
</style>
