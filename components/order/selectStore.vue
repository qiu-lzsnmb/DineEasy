<template>
  <view>
    <view class="search-input">
      <uni-search-bar
        @input="inputend"
        cancelButton="none"
        clearButton="none"
        placeholder="请输入店名或者地址"
        :radius="100" />
    </view>
    <view class="store-list">
      <view
        class="store-item"
        @click="selectStore(item)"
        :class="selectId == item.id ? 'select' : ''"
        v-for="item in storeDataRef"
        :key="item.id">
        <view class="store-item-info">
          <view class="name">{{ item.name }}</view>
          <view class="type">
            <view v-if="item.status == 0">
              <uni-tag
                text="停业中"
                type="default"
                inverted="true"
                size="mini" />
            </view>
            <view v-if="!isItOpen(item.start_time, item.end_time)">
              <uni-tag
                text="打样中"
                type="error"
                inverted="true"
                size="mini" />
            </view>
            <view
              v-else
              style="display: flex">
              <view v-if="[1, 4, 5, 6].includes(item.status)">
                <uni-tag
                  text="可堂食"
                  type="success"
                  inverted="true"
                  size="mini" />
              </view>
              <view v-if="[2, 4, 5, 7].includes(item.status)">
                <uni-tag
                  text="可自提"
                  type="primary"
                  inverted="true"
                  size="mini" />
              </view>
              <view v-if="[3, 4, 6, 7].includes(item.status)">
                <uni-tag
                  text="可外卖"
                  type="warning"
                  inverted="true"
                  size="mini" />
              </view>
            </view>
          </view>
          <view class="address">
            <uni-icons
              custom-prefix="iconfont"
              type="icon-wxbdingwei"
              size="20" />
            {{ item.address }}
          </view>
          <view class="startTime">
            <uni-icons
              custom-prefix="iconfont"
              type="icon-shijian"
              size="20" />
            {{ hToMinute(item.start_time) }}-{{ hToMinute(item.end_time) }}
          </view>
        </view>
        <view class="store-item-operate">
          <view>去点单</view>
          <view class="jl">距离{{ item.distance }}米</view>
          <view class="operate">
            <view @click.stop="callPhone(item.phone)">
              <uni-icons
                color="#f0ad4e"
                custom-prefix="iconfont"
                type="icon-dianhuazhengzaibohao"
                size="23" />
            </view>
            <view @click.stop="toAddress(item.address)">
              <uni-icons
                color="#f0ad4e"
                custom-prefix="iconfont"
                type="icon-daohang"
                size="23" />
            </view>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script setup>
  import { ref } from 'vue';
  import api from '@/api';
  import { onLoad, onPullDownRefresh } from '@dcloudio/uni-app';
  import { isItOpen, hToMinute } from '@/utils/index.js';
  const storeData = uni.getStorageSync('Store');
  const selectId = ref(storeData.id);
  const storeDataRef = ref([]);
  let inputTime = null;

  //搜索门店
  function search(val) {
    console.log(val);
  }

  //输入完成，采用防抖
  function inputend(val) {
    if (inputTime != null) {
      clearTimeout(inputTime);
    }
    inputTime = setTimeout(() => {
      search({
        value: val
      });
    }, 600);
  }

  function selectStore(item) {
    if (item == 0) {
      uni.showToast({
        title: '该门店已闭店!',
        icon: 'error',
        duration: 2000
      });
      return;
    } else if (!isItOpen(item.start_time, item.end_time)) {
      uni.showToast({
        title: '该门店不在营业哦!',
        icon: 'error',
        duration: 2000
      });
      return;
    }
    if (item.id == selectId.value) {
      uni.setStorageSync('Store', { id: item.id, name: item.name, address: item.address });
      uni.$emit('selectOK');
    }
    selectId.value = item.id;
  }

  async function getStores(pagenum, pagesize) {
    const res = await api.store.getStores(pagenum, pagesize);
    if (res.code == 200) {
      storeDataRef.value.push(...res.data);
      return true;
    }
    uni.showToast({
      title: '请检查网络!!',
      icon: 'none'
    });
    return false;
  }
  function callPhone(num) {
    uni.makePhoneCall({
      phoneNumber: num,
      success: res => {
        console.log(res);
      },
      fail: err => {
        console.log(err);
      },
      complet: result => {
        console.log(result);
      }
    });
  }
  function toAddress(address) {
    uni.openLocation({
      latitude: 27.999081, //目标纬度
      longitude: 120.692603, //目标经度
      name: address, //名称
      address: address, //地址
      scale: 28
    });
  }

  onLoad(() => {
    //状态 0打样，1可堂食，2可自提，3可外卖
    getStores(-1, 6);
  });
  onPullDownRefresh(() => {
    storeDataRef.value = [];
    if (getStores(-1, 6)) {
      uni.stopPullDownRefresh();
    }
  });
</script>

<style lang="scss">
  .store-list {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
  }

  .store-item {
    height: 10rem;
    border: 2px solid #e2e2e2;
    width: 90%;
    padding: 1.1rem;
    padding-right: 0px;
    box-sizing: border-box;
    margin-bottom: 1rem;
    border-radius: 0.7rem;
    display: flex;
    justify-content: space-between;
  }

  .store-item-info {
    .name {
      font-size: 1.2rem;
      font-weight: 600;
    }

    .type {
      display: flex;

      view {
        margin: 3px;
      }
    }

    .address,
    .startTime {
      font-size: 0.7rem;
      display: flex;
      align-items: center;
    }
  }

  .store-item-operate {
    width: 38%;
    display: flex;
    align-items: center;
    flex-direction: column;
    border-left-style: solid;
    border-left-width: 1px;
    border-left-color: #c3c3c3;
    font-size: 0.9rem;
    padding: 1.1rem;
    box-sizing: border-box;
    justify-content: space-between;

    .jl {
      font-size: 0.7rem;
    }

    .operate {
      display: flex;
      width: 100%;

      justify-content: space-around;

      view {
        display: flex;
        align-items: center;
        justify-content: center;
        background-color: rgb(234, 234, 234);
        width: 36px;
        height: 36px;
        border-radius: 50%;
      }
    }
  }

  .select {
    border: 3px solid $theme-color;
    position: relative;
    box-shadow: 0px 2px 7px 0px rgba(85, 110, 97, 0.35);

    &:before {
      content: '';
      position: absolute;
      right: 0;
      bottom: 0;
      border: 16px solid $theme-color;
      border-top-color: transparent;
      border-left-color: transparent;
    }

    &:after {
      content: '';
      width: 4px;
      height: 14px;
      position: absolute;
      right: 6px;
      bottom: 3px;
      border: 2px solid #fff;
      border-top-color: transparent;
      border-left-color: transparent;
      transform: rotate(45deg);
    }
  }
</style>
