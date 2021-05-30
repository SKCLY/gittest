/**
* 院校优先
* @auth junzilan
*/
<template>
  <view class="content">
    <view class="fixdTop">
      <view class="bartitles">
        <view class='topLeftV' @click="jumpSearch">
          <image style="width: 34rpx;height: 34rpx;margin-left: 12rpx;"
            src="https://img.caochangjihe.com/miniapp/volunteer/icon_search.png"></image>
          <view class='input'>请输入大学</view>
        </view>
      </view>
      <!-- 分类bar -->
      <view class="categoryView">
        <view class="categoryViewTitle" @click="jumpAlert('PROVINCE')">
          <text style="left: 0;">省份</text>
          <view class="pointView" v-if="requestParmas.provinceIds!=''" />
          <image class="categoryViewTitleimage"
            src="https://img.caochangjihe.com/miniapp/volunteer/intenlligent/icon_arrow_up.png"></image>
          <u-popups v-model="showProvinceAlert" mode="bottom" border-radius="24">
            <normalDialog @commonDialogConfirm="commonDialogConfirm" selectype="Multiple" title="省份"
              :isShow="showProvinceAlert" @closeDialog="closeDialog" />
          </u-popups>
        </view>
        <view class="categoryViewTitle" @click="jumpAlert('ARRANGE')">
          <text style="left: 0;">层次</text>
          <view class="pointView" v-if="requestParmas.schoolKind!=''||requestParmas.tags!='' " />
          <image class="categoryViewTitleimage"
            src="https://img.caochangjihe.com/miniapp/volunteer/intenlligent/icon_arrow_up.png">
          </image>
          <u-popups v-model="showArrangeAlert" mode="bottom" border-radius="24">
            <arrangementDialog @arrangeDialogConfirm="arrangeDialogConfirm" @closeDialog="closeDialog"
              :isShow="showArrangeAlert" />
          </u-popups>
        </view>
        <view class="categoryViewTitle" @click="jumpAlert('ORDER')">
          <text style="left: 0;">排序</text>
          <view class="pointView" v-if="requestParmas.sortType!=2" />
          <image class="categoryViewTitleimage"
            src="https://img.caochangjihe.com/miniapp/volunteer/intenlligent/icon_arrow_up.png">
          </image>
          <u-popups v-model="showOrderAlert" mode="bottom" border-radius="24">
            <orderDialog @OrderDialogConfirm="OrderDialogConfirm" :idsEmpty="idsEmpty" :isShow="showOrderAlert"
              showType="INSTITU" @closeDialog="closeDialog" />
          </u-popups>
        </view>
        <view class="categoryViewTitle" @click="jumpAlert('TYPE')">
          <text style="left: 0;">类型</text>
          <view class="pointView" v-if="requestParmas.schoolType!=''" />
          <image class="categoryViewTitleimage"
            src="https://img.caochangjihe.com/miniapp/volunteer/intenlligent/icon_arrow_up.png">
          </image>
          <u-popups v-model="showTypeAlert" mode="bottom" border-radius="24">
            <normalDialog @commonDialogConfirm="commonDialogConfirm" selectype="Multiple" title="类型"
              @closeDialog="closeDialog" :isShow="showTypeAlert" />
          </u-popups>
        </view>
      </view>

    </view>
    <!-- 列表 -->
    <view class="listView">
      <volunteerListView :lists="lists" itemType="COLLEGE" @itemClick="showSchoolDialog"></volunteerListView>
      <view class="vipCard" v-if="!isAllowLoadMore">
        <view class="vipCard_title">开通VIP或加入所选省份对应学校的班级</view>
        <view class="vipCard_tip">开通VIP可查看全部推荐院校</view>
        <view class="openBtn">
          <image class="openBtn_left" @click="jumpOpenVip()"
            src="https://img.caochangjihe.com/miniapp/volunteer/intenlligent/icon_openvip.png">
          </image>
          <view class="openBtn_right" @click="jumpAddClasss()">加入班级</view>
        </view>
      </view>
    </view>

    <view class="emptyList" v-if="isAllowLoadMore&&lists.length==0">
      <image src="https://img.caochangjihe.com/miniapp/volunteer/xs_empty.png" style="width: 274rpx;height: 274rpx;">
      </image>
      <view>未匹配到相关数据～</view>
    </view>

    <!-- 底部view -->
    <view class="content_bottom">
      <view class="showText">
        <view style="display: flex;flex-direction: row;align-items: center;" @click="jumpEdit">
          <view>{{information.score}}分</view>
          <view style="margin-left: 16rpx;">{{information.showTextInfo}}</view>
          <image src="https://img.caochangjihe.com/miniapp/volunteer/intenlligent/icon_edit.png"
            style="width: 32rpx;height: 32rpx;margin-left: 4rpx;">
          </image>
        </view>
        <view style="display: flex;flex-direction: row;align-items: center;" @click="jumpEditBatch">
          <view style="margin-left: 16rpx;">{{information.batch}}</view>
          <image src="https://img.caochangjihe.com/miniapp/volunteer/intenlligent/icon_edit.png"
            style="width: 32rpx;height: 32rpx;margin-left: 4rpx;">
          </image>
        </view>
      </view>
      <view class="priview" v-if="tempLength>0" @click="jumpPrview">
        预览志愿表({{tempLength}})</view>
      <view class="priviews" v-else>预览志愿表</view>
    </view>

    <view>
      <u-popup border-radius="24" v-model="schoolDialogData.show" :mode="'bottom'" length="80%" :mask="true"
        :closeable="true" :close-icon-pos="'top-right'">
        <schoolDialog :param="schoolDialogData.param" :type="'school'" :titleName="schoolDialogData.schoolName"
          @buttonClick="goSchoolDetail"></schoolDialog>
      </u-popup>
    </view>

  </view>
</template>
<script>
  import volunteerListView from "@/components/volunteer/volunteerListView.vue";
  import arrangementDialog from "@/components/volunteer/arrangementDialogView.vue";
  import normalDialog from "@/components/volunteer/normalDialogView.vue";
  import orderDialog from "@/components/volunteer/orderDialogView.vue";
  import screenDialog from "@/components/volunteer/screenDialogView.vue";
  import volunteerApi from "@/services/api/volunteerApi.js";
  import schoolDialog from "@/components/volunteer/majorSchoolDialog.vue";

  export default {
    components: {
      orderDialog,
      screenDialog,
      normalDialog,
      arrangementDialog,
      volunteerListView,
      schoolDialog
    },
    data() {
      return {
        showProvinceAlert: false, //省份
        showArrangeAlert: false, //层次
        showOrderAlert: false, //排序
        showTypeAlert: false, //类型
        requestParmas: {
          pageSize: 20,
          page: 1,
          provinceIds: '', // 所选省份id
          schoolType: "", // 院校类型 综合、理工
          tags: '', // 院校标签
          schoolKind: "", // 办学性质 985 211
          sortType: 2, // 排序方式 0概率，1招生人数，2大学排名，3按照匹配专业数
        },
        lists: [],
        isAllowLoadMore: true,
        selctPopupType: 1,
        information: {
          showTextInfo: '',
        },
        schoolDialogData: {
          show: false,
          schoolName: '',
          param: {}
        },
        tempPreferenceBean: {},
        isFirst: true,
        idsEmpty: false,
        tempLength: 0
      }
    },
    onLoad() {
      this.getTempList()
    },
    onShow() {
      this.getScore()
      if (!this.isFirst) {
        this.getShowTempList()
      }
    },
    onReachBottom() {
      if (this.isAllowLoadMore) {
        let parmas = this.requestParmas
        parmas.page++
        this.getData(parmas)
      }
    },
    onPullDownRefresh() {
      let parmas = this.requestParmas
      parmas.page = 1
      this.getData(parmas)
      this.getScore()
      setTimeout(function() {
        uni.stopPullDownRefresh();
      }, 1000);
    },
    onShareAppMessage() {
      return {
        title: '我正在使用【自涞志愿填报助手】',
        imageUrl: 'https://img.caochangjihe.com/miniapp/share/content-bg.png',
        path: '/pages/volunteer'
      }
    },
    methods: {
      jumpEdit() {
        uni.navigateTo({
          url: "/pages/volunteer/artsSciences/index?isEdit=true"
        })
      },
      jumpEditBatch() {
        uni.navigateTo({
          url: "/pages/volunteer/batchInfo/index?isEdit=true"
        })
      },
      showSchoolDialog(event) {
        //学校弹窗
        this.schoolDialogData = {
          show: true,
          schoolName: event.data.schoolName,
          param: {
            matchPercent: event.data.matchPercent,
            schoolCode: event.data.schoolCode
          }
        }
      },
      goSchoolDetail(data) {
        //跳转到学校详情
        uni.navigateTo({
          url: "/pages/volunteer/findSchool/schoolDetail/schoolDetail?schoolCode=" + data.code,
        });
      },
      //跳转搜索
      jumpSearch() {
        if (this.isAllowLoadMore) {
          uni.navigateTo({
            url: "../search/search_univeris"
          })
        } else {
          uni.showModal({
            content: '开通VIP，才可搜索',
            confirmText: "去开通",
            cancelColor: "#000000",
            confirmColor: "#576B95",
            cancelText: "取消",
            success: function(res) {
              if (res.confirm) {
                uni.navigateTo({
                  url: "/pages/volunteer/vip/index",
                })
              } else if (res.cancel) {
                console.log('用户点击取消');
              }
            }
          });
        }
      },
      //弹窗
      jumpAlert(type) {
        if (type == "PROVINCE") { //省份
          this.showProvinceAlert = !this.showProvinceAlert
          this.selctPopupType = 1
        } else if (type == "ARRANGE") { //层次
          this.showArrangeAlert = !this.showArrangeAlert
        } else if (type == "ORDER") { //排序
          this.showOrderAlert = !this.showOrderAlert
        } else if (type == "TYPE") { //类型
          this.showTypeAlert = !this.showTypeAlert
          this.selctPopupType = 2
        }

      },
      //预览
      jumpPrview() {
        if (this.isAllowLoadMore) {
          uni.navigateTo({
            url: '/pages/volunteer/examinationCenter/preview/preview?type=INSTITU'
          })
        } else {
          uni.showModal({
            content: '开通VIP，才可查看志愿表',
            confirmText: "去开通",
            cancelColor: "#000000",
            confirmColor: "#576B95",
            cancelText: "取消",
            success: function(res) {
              if (res.confirm) {
                uni.navigateTo({
                  url: "/pages/volunteer/vip/index",
                })
              } else if (res.cancel) {
                console.log('用户点击取消');
              }
            }
          });
        }
      },
      jumpOpenVip() {
        uni.navigateTo({
          url: "/pages/volunteer/vip/index",
        })
      },
      jumpAddClasss() {
        uni.showModal({
          content: '该功能需下载app才可使用，是否去下载？',
          confirmText: "去下载",
          cancelColor: "#000000",
          confirmColor: "#576B95",
          cancelText: "取消",
          success: function(res) {
            if (res.confirm) {
              uni.navigateTo({
                url: "/pages/webView/downloadApp/downloadApp",
              })
            } else if (res.cancel) {
              console.log('用户点击取消');
            }
          }
        });
      },
      //-------------------------------------网络请求---------------------------------------------------------------
      getData(parmas) {
        var that = this
        this.$https(volunteerApi.intelligentFilling, 'GET', parmas).then(res => {
          if (res.success) {
            var responListOld = res.data.recommendInfos
            var responList = that.formatData(responListOld)
            if (that.requestParmas.page > 1) {
              if (responList.length > 0) {
                that.lists = that.lists.concat(responList)
              } else {
                that.requestParmas.page--
              }
            } else {
              that.lists = responList
              that.isAllowLoadMore = res.data.vip
            }
          } else {
            uni.showToast({
              title: res.message,
              icon: "none",
              duration: 2000
            });
          }
        })
      },
      formatData(list) {
        var type = this.tempPreferenceBean.type
        var tempList = this.tempPreferenceBean.groups
        for (var i = 0; i < list.length; i++) {
          var str = "" //名称 // 1 老高考, 2:新高考
          list[i].showTempText = ""
          for (var i1 = 0; i1 < tempList.length; i1++) {
            if (tempList[i1].schoolCode == list[i].schoolCode) {
              if (str == "") {
                str = str + (type == 1 ? String.fromCharCode(65 + tempList[i1].groupId) : tempList[i1].groupId)
              } else {
                str = "," + str + (type == 1 ? String.fromCharCode(65 + tempList[i1].groupId) : tempList[i1].groupId)
              }
            }
            if (str != "") {
              list[i].showTempText = str + "志愿"
            }
          }
        }
        return list
      },
      getTempList() {
        var that = this
        var types =
          this.$https('/userPreference/getTempPreference', 'GET', {}).then(res => {
            that.isFirst = false
            that.tempPreferenceBean = res.data
            that.tempLength = res.data.groups.length
            let parmas = that.requestParmas
            parmas.page = 1
            that.getData(parmas)
          })
      },
      //志愿-获取我的临时志愿预览
      getShowTempList() {
        var that = this
        this.$https('/userPreference/getTempPreference', 'GET', {}).then(res => {
          that.tempPreferenceBean = res.data
          that.tempLength = res.data.groups.length
          var list1 = that.lists
          if (list1.length > 0) {
            that.lists = that.formatData(list1)
          }
        })
      },
      getScore() {
        var that = this
        this.$https('/userExamInfo/getUserExamInfoByUserId', 'GET', {}).then(res => {
          that.information = res.data
          var str = ""
          if (res.data.subjectNames) {
            for (var i = 0; i < res.data.subjectNames.length; i++) {
              if (i != res.data.subjectNames.length - 1) {
                str = str + res.data.subjectNames[i] + "/"
              } else {
                str = str + res.data.subjectNames[i]
              }
            }
          }
          that.information.showTextInfo = str
        })
      },

      //---------------------------------------------回调---------------------------------------------------------------
      //公共弹窗页面确认返回 省份返回id列表&类型弹窗
      commonDialogConfirm(e) {
        console.log("省份的回调:" + e.selectList)
        if (this.selctPopupType == 1) { //省份
          let list = e.selectList
          var list2 = []
          for (var i = 0; i < list.length; i++) {
            list2.push(list[i].id)
          }
          let parmas = this.requestParmas
          parmas.page = 1
          parmas.provinceIds = list2.toString()
          this.getData(parmas)
        } else {
          let list = e.selectList
          var list2 = []
          for (var i = 0; i < list.length; i++) {
            list2.push(list[i].id)
          }
          let parmas = this.requestParmas
          parmas.page = 1
          parmas.schoolType = list2.toString()
          this.getData(parmas)
        }
        this.scrollTop()
      },
      scrollTop() {
        uni.pageScrollTo({
          duration: 0, //过渡时间必须为0，否则运行到手机会报错
          scrollTop: 0 //滚动到实际距离是元素距离顶部的距离减去最外层盒子的滚动距离（如res.top - data.top）
        })
      },
      //层次点击的回调
      arrangeDialogConfirm(e) {
        console.log("层次点击的回调:" + e)
        let parmas = this.requestParmas
        parmas.page = 1
        var signlelist = e.signlelist //办学性质
        if (signlelist.length > 0) {
          parmas.schoolKind = signlelist[0].id
        }

        var multiplelist = e.multiplelist //院校标签 985 211
        var list = []
        for (var i = 0; i < multiplelist.length; i++) {
          list.push(multiplelist[i].id)
        }
        parmas.tags = list.toString()
        this.getData(parmas)
        this.scrollTop()
      },
      //排序弹窗的回调
      OrderDialogConfirm(e) {
        console.log("排序弹窗的回调:" + e.selectList)
        let list = e.selectList
        let parmas = this.requestParmas
        parmas.page = 1
        parmas.sortType = list.toString()
        this.getData(parmas)
        this.scrollTop()
      },
      //关闭弹窗
      closeDialog(e) {
        this.showProvinceAlert = false, //省份
          this.showArrangeAlert = false, //层次
          this.showOrderAlert = false, //排序
          this.showTypeAlert = false //类型
      }
    },
  }
</script>

<style>
  page {
    background-color: #F7F7F8;
  }

  .content {
    display: flex;
    width: 100%;
    background: #F7F7F8;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding-bottom: 140rpx;
  }

  .fixdTop {
    display: flex;
    flex-direction: column;
    position: relative;
    position: fixed;
    top: 0;
    z-index: 9;
    width: 100%;
  }

  .listView {
    margin-top: 188rpx;
    position: relative;
    width: 100%;
  }

  .bartitles {
    width: 100%;
    display: flex;
    padding-top: 20rpx;
    flex-direction: column;
    background: #FFFFFF;
    padding-bottom: 20rpx;
  }

  .topLeftV {
    height: 68rpx;
    padding-left: 12rpx;
    padding-right: 12rpx;
    margin-right: 32rpx;
    margin-left: 32rpx;
    border-radius: 34rpx;
    background: #F7F7F8;
    display: flex;
    flex-direction: row;
    position: relative;
    justify-content: space-between;
    align-items: center;
  }

  .input {
    padding-left: 12rpx;
    font-family: PingFangSC-Regular;
    font-size: 26rpx;
    color: #B5B5B5;
    flex: 1;
  }

  .categoryView {
    width: 100%;
    display: flex;
    flex-direction: row;
    align-items: center;
    height: 80rpx;
    background-color: #F7F7F8;
    padding-left: 32rpx;
    padding-right: 32rpx;
    justify-content: space-between;
  }

  .categoryView>view {
    font-size: 24rpx;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #333333;
  }

  .content_bottom {
    display: flex;
    width: 100%;
    flex-direction: row;
    align-items: center;
    padding-left: 32rpx;
    padding-right: 32rpx;
    height: 136rpx;
    position: fixed;
    background: #FFFFFF;
    justify-content: space-between;
    bottom: 0;
    left: 0;
  }

  .showText {
    display: flex;
    flex-direction: row;
    align-items: center;
    font-size: 24rpx;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #333333;
    line-height: 17px;
  }

  .priview {
    width: 272rpx;
    height: 72rpx;
    background: #FEBD02;
    border-radius: 36rpx;
    font-size: 28rpx;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #333333;
    line-height: 20px;
    justify-content: center;
    display: flex;
    align-items: center;
  }

  .priviews {
    width: 272rpx;
    height: 72rpx;
    background: #F7F7F8;
    border-radius: 36rpx;
    font-size: 28rpx;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #8C8C8C;
    line-height: 20px;
    justify-content: center;
    display: flex;
    align-items: center;
  }

  .vipCard {
    width: 91.4%;
    height: 244rpx;
    margin: 16rpx auto;
    background: linear-gradient(315deg, #FFFFFF 0%, #FFFAF0 100%);
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .vipCard_title {
    font-size: 32rpx;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #333333;
  }

  .vipCard_tip {
    margin-top: 4rpx;
    margin-bottom: 24rpx;
    font-size: 28rpx;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #333333;
  }

  .openBtn {
    width: 100%;
    padding-left: 82rpx;
    padding-right: 82rpx;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
  }

  .openBtn_left {
    width: 180rpx;
    height: 64rpx;
  }

  .openBtn_right {
    width: 180rpx;
    height: 64rpx;
    border-radius: 32rpx;
    border: 1px solid #FEBD02;
    font-size: 28rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: PingFangSC-Medium, PingFang SC;
    font-weight: 500;
    color: #FEBD02;
  }

  .categoryViewTitle {
    position: relative;
    padding-right: 32rpx;
  }

  .categoryViewTitleimage {
    top: 2px;
    position: absolute;
    width: 24rpx;
    height: 24rpx;
    margin-left: 8rpx
  }

  .emptyList {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    font-size: 28rpx;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
    color: #939599;
    position: absolute;
    z-index: 2;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -60%);
  }

  .pointView {
    width: 12rpx;
    height: 12rpx;
    background: #FF4242;
    border-radius: 12rpx;
    position: absolute;
    left:57%;
     top:-3px;
  }
</style>
