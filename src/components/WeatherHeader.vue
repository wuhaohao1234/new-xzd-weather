<script setup>
import { defineProps, onMounted, ref } from "vue";
const props = defineProps({
  cityInfo: {
    type: Object,
    default: null,
  },
});
const emits = defineEmits(['handleSave']);
const save = () => {
  emits('handleSave')
}

const adcode = ref(null);
const isSave = ref(false)
const showToast = ref(false)
onMounted(() => {
  adcode.value = window.location.search.split("=")[1];
  const arr = JSON.parse(localStorage.getItem('cityList'))
  if(!arr) {
    isSave.value = true
    return
  }
  isSave.value = arr?.findIndex(item => item.adcode === adcode.value) === -1
  
})
</script>

<template>
  <div class="weather-header">
    <div class="nav">
      <div class="nav-left">
        <a
          href="/weather"
          class="router-link-active router-link-exact-active"
          aria-current="page"
        >
          <i class="fa-solid fa-sun text-2xl"></i>
          新中地天气
        </a>
        <h4 class="text-lg">{{ cityInfo?.city || "" }}</h4>
        <span class="text-sm"
          >实时天气：{{ cityInfo?.weather }} {{ cityInfo?.temperature }}℃</span
        ><span class="text-sm"
          >{{ cityInfo?.winddirection }}风{{ cityInfo?.windpower }}级</span
        >
      </div>

      <div class="flex gap-3 flex-1 justify-end">
        <i
          class="fa-solid fa-circle-info text-xl hover:text-weather-secondary cursor-pointer duration-1000"
          style="margin-right: .75rem;cursor: pointer;"
          @click="showToast = true"
        ></i>
        <i
          @click="save"
          v-show="adcode && isSave"
          class="fa-solid fa-plus text-xl hover:text-weather-secondary duration-150 cursor-pointer"
          style="margin-right: .75rem;cursor: pointer;"
        ></i>
        <!---->
      </div>
      <div class="toast" v-show="showToast" >
        <div class="toast-content">
          <h2>关于: </h2>
          <p style="font-size: .75rem;margin-bottom: 1rem;line-height: 1rem;" class="text-xs mb-4">这个应用可以用来追踪你选择城市的当前天气</p>
          <h2 >如何使用:</h2>
          <p style="font-size: .75rem;margin-bottom: 1rem;line-height: 1rem;" > 1.点击搜索框输入你希望追踪的城市<br> 2.在搜索结果中选中一个城市，你将获取当地最新的天气<br> 3.点击右侧的＋号可以将追踪城市的天气情况保存在首页 </p>
          <h2>移除城市:</h2>
          <p style="font-size: .75rem;margin-bottom: 1rem;line-height: 1rem;" > 如果你不想在首页关注某个城市,可以通过底部的按钮删<br>除它 </p>
          <div class="close" @click="showToast = false" >关闭</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.weather-header {
  height: 80px;
  background-color: rgb(0 102 138);
  --tw-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1),
    0 4px 6px -4px rgb(0 0 0 / 0.1);
  --tw-shadow-colored: 0 10px 15px -3px var(--tw-shadow-color),
    0 4px 6px -4px var(--tw-shadow-color);
  box-shadow: var(--tw-ring-offset-shadow, 0 0 #0000),
    var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow);
}
.nav {
  width: 960px;
  margin: 0 auto;
  padding: 1.5rem 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #fff;
}

a {
  color: #fff;
  text-decoration: none;
  font-size: 1.5rem;
  width: 156px;
}

h4 {
  font-size: 1.125rem;
  margin-left: 1rem;
}
.text-sm {
  font-size: 0.875rem;
  margin-left: 1rem;
}
.gap-3 {
  margin-left: 0.75rem;
  justify-content: flex-end;
}
.nav-left {
  display: flex;
  align-items: center;
}
.toast {
  width: 100vw;
  height: 100vh;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 99;
}
.toast-content {
  width: 298px;
  box-sizing: content-box;
  background: #fff;
  margin: 128px auto;
  color: #000;
  padding: 1rem;
  font-size: .75rem;
}
.toast-content h2 {
  font-size: 16px;
  font-weight: 400;
}
.close {
  color: rgb(255 255 255);
  background: rgb(0 102 138);
  display: inline-block;
  padding: .5rem 1.5rem;
  cursor: pointer;
}
</style>
