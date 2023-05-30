<script setup>
import { defineProps, onMounted, ref, watch } from "vue";
import { Line } from "@antv/g2plot";

const transformedData = ref([]);
const inputCity = ref("");
const newCityInfo = ref(null);
const localCity = ref([]);
const searchCityInfo = ref(null);
const props = defineProps({
  casts: {
    type: Array,
    default: null,
  },
});
const emits = defineEmits(["changeNewCityInfo"]);
const adcode = ref(null);
const newCasts = ref(null);
onMounted(() => {
  adcode.value = window.location.search.split("=")[1];
  const key = "6806a666fdfcd4bb5cac88647eb9dc34";
  if (adcode.value) {
    fetch(
      `https://restapi.amap.com/v3/weather/weatherInfo?key=${key}&city=${adcode.value}`
    )
      .then((res) => res.json())
      .then((result) => {
        newCityInfo.value = {
          city: result.lives[0].city,
          weather: result.lives[0].weather,
          temperature: result.lives[0].temperature,
          winddirection: result.lives[0].winddirection,
          windpower: result.lives[0].windpower,
          adcode: result.lives[0].adcode,
        };
        emits("changeNewCityInfo", newCityInfo.value);
        console.log(newCityInfo.value);
      });

    fetch(
      `https://restapi.amap.com/v3/weather/weatherInfo?key=${key}&city=${adcode.value}&extensions=all`
    )
      .then((res) => res.json())
      .then((result) => {
        newCasts.value = result.forecasts[0].casts;
      });
  } else {
    localCity.value = JSON.parse(localStorage.getItem("cityList"))?.map(item => {item.isHover = false; return item});
    console.log(localCity.value);
  }
});
const searchCity = () => {
  if (inputCity.value === "") {
    return;
  }
  fetch(
    `http://restapi.amap.com/v3/geocode/geo?key=6806a666fdfcd4bb5cac88647eb9dc34&address=${inputCity.value}`
  )
    .then((res) => res.json())
    .then((result) => {
      if (result?.geocodes[0].city.length === 0) {
        return;
      }
      searchCityInfo.value = result;
      console.log(searchCityInfo.value);
      // window.open(`http://localhost:5173/?adcode=${result.geocodes[0].adcode}`)
    });
};
const getWeek = (key, week) => {
  if (key === 0) {
    return "今天";
  }
  if (key === 1) {
    return "明天";
  }
  week = +week;
  switch (week) {
    case 1:
      return "周一";
    case 2:
      return "周二";
    case 3:
      return "周三";
    case 4:
      return "周四";
    case 5:
      return "周五";
    case 6:
      return "周六";
    case 7:
      return "周日";
  }
};
const openNewWindow = (adcode) => {
  window.open(`http://localhost:5173/?adcode=${adcode}`);
};
watch(
  () => props.casts,
  (newValue) => {
    if (newCasts.value) {
      transformedData.value = newCasts.value.flatMap(
        ({ Date, daytemp, nighttemp }, key) => [
          { Date: key, temp: parseInt(daytemp), type: "day" },
          { Date: key, temp: parseInt(nighttemp), type: "night" },
        ]
      );
    } else {
      transformedData.value = newValue.flatMap(
        ({ Date, daytemp, nighttemp }, key) => [
          { Date: key, temp: parseInt(daytemp), type: "day" },
          { Date: key, temp: parseInt(nighttemp), type: "night" },
        ]
      );
    }

    const line = new Line("container", {
      data: transformedData.value,
      padding: "auto",
      xField: "Date",
      yField: "temp",
      seriesField: "type",
      tooltip: false,
      legend: false,
      label: {
        visible: true, // 显示数值
        style: {
          fill: "#ffffff", // 设置字体颜色为白色
        },
        formatter: (text, item) => {
          // 自定义 label 文字内容
          return `${item._origin.type === "day" ? "白" : "晚"}  ${
            item._origin.temp
          }℃`;
          // return item.daytemp
        },
      },
      reflect: "y",
      xAxis: false, // 隐藏横轴
      yAxis: false, // 隐藏纵轴
      point: true,
    });
    line.render();
  }
);
const deleteLocal = (localCity, key) => {
  if(localCity.length === 1) {
    localCity.splice(key, 1)
    localStorage.setItem('cityList', null)
    return
  }
  localStorage.setItem('cityList', JSON.stringify(localCity.splice(key, 1)))
}
</script>

<template>
  <div class="weather-content">
    <input
      type="text"
      v-show="!newCasts"
      v-model="inputCity"
      @keydown="searchCity"
      placeholder="请输入城市名称"
    />
    <div
      class="header-city"
      v-show="searchCityInfo"
      style="cursor: pointer"
      @click="openNewWindow(searchCityInfo?.geocodes[0].adcode)"
    >
      <div class="header-city-name">{{ searchCityInfo?.geocodes[0].city }}</div>
    </div>
    <div
      v-for="(item, key) in localCity"
      :key="key"
      :class="item.isHover ? 'header-city-animation' : 'header-city-animation'"
      @mousemove="item.isHover = true"
      @mouseleave="() => {
          item.isHover = false
      }"
    >
      <div :class="item.isHover ? 'header-city-hover' : 'header-city'" v-show="localCity.length > 0">
        <div class="header-city-name">{{ item.city }}</div>
        <div class="header-city-team">{{ item.temperature }}度</div>
      </div>
      <div class="button" @click="()=> {
        openNewWindow(item.adcode)
        
      }" >查看</div>
      <div class="button" @click="deleteLocal(localCity,key)" >删除</div>
    </div>

    <h2 v-show="!newCityInfo">近期天气</h2>
    <div v-show="newCityInfo" style="text-align: center" class="new-city-info">
      <h2>
        你正在预览{{
          newCityInfo?.city
        }}的天气信息，可以通过右上角的"+"号按钮保存起来
      </h2>
      <span>当日气温是：{{ newCityInfo?.temperature }}摄氏度</span>
      <span>当日天气是：{{ newCityInfo?.weather }}</span>
      <span> 当日风向是：{{ newCityInfo?.winddirection }}风 </span>
      <span> 当日风力是：{{ newCityInfo?.windpower }}级 </span>
    </div>

    <div class="weather-result">
      <div class="flex gap-6">
        <div
          v-show="newCasts"
          class="flex flex-col flex-1 text-center gap-4"
          v-for="(item, key) in newCasts"
          :key="key"
        >
          <span>{{ getWeek(key, item.week) }}</span
          ><span>{{ item.date }}</span
          ><span>{{ item.dayweather }}</span
          ><span>风力 {{ item.nightpower }}级</span>
        </div>
        <div
          v-show="!newCasts"
          class="flex flex-col flex-1 text-center gap-4"
          v-for="(item, key) in casts"
          :key="key"
        >
          <span>{{ getWeek(key, item.week) }}</span
          ><span>{{ item.date }}</span
          ><span>{{ item.dayweather }}</span
          ><span>风力 {{ item.nightpower }}级</span>
        </div>
      </div>
      <div
        id="container"
        style="
          width: 864px;
          height: 160px;
          user-select: none;
          -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
          padding: 0px;
          margin: 0px;
          border-width: 0px;
          display: block;
        "
      ></div>
    </div>
  </div>
</template>

<style scoped>
.weather-content {
  width: 960px;
  overflow: scroll;
  /* background: #000; */
  color: #fff;
  margin: 0 auto;
  padding-top: 1rem;
}

input {
  width: 100%;
  padding: 0.5rem 0.25rem;
  background: transparent;
  border-color: #e5e7eb;
  border-width: 0;
  border-bottom-width: 1px;
  color: inherit;
  font-size: 100%;
}

input:active,
input:focus {
  outline: 2px solid transparent;
  border-width: 0;
  border-color: #e5e7eb;
  outline: 0;
  outline-offset: 2px;
  --tw-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --tw-ring-offset-shadow: 0 0 #0000;
  --tw-shadow-colored: 0 4px 6px -1px var(--tw-shadow-color),
    0 2px 4px -2px var(--tw-shadow-color);
  box-shadow: var(--tw-ring-offset-shadow, 0 0 #0000),
    var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow);
}

h2 {
  margin-top: 24px;
  font-size: 16px;
  line-height: 24px;
}

.weather-result {
  background: rgb(0 78 113);
  margin-top: 8px;
  padding: 2.5rem 3rem;
  border-bottom: 0;
}

.gap-6 {
  display: flex;
  justify-content: space-around;
}

span {
  display: block;
  width: 134px;
  height: 24px;
  line-height: 24px;
  text-align: center;
  margin-bottom: 1rem;
}
.header-city {
  background: rgb(0 78 113);
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex: auto;
  width: 100%;
  flex: none;
  line-height: 24px;
  padding: 8px 16px;
  cursor: pointer;

  transition: width 0.3s ease-in-out;
}
.header-city-hover {
  background: rgb(0 78 113);
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex: auto;
  width: 80%;
  flex: none;
  line-height: 24px;
  padding: 8px 16px;
  cursor: pointer;

  transition: width 0.3s ease-in-out;
}
.header-city-animation {
  display: flex;
  position: relative;
  align-items: center;
  width: 100%;
  margin-top: 1rem;
}


.new-city-info h2 {
  margin-bottom: 1rem;
  background: rgb(0 78 113);
  padding: 0.5rem;
}
.new-city-info span {
  display: block;
  text-align: center;
  width: 100%;
  margin: 0;
  margin-bottom: 1rem;
}
.button {
  text-align: center;
  background: rgb(234 179 8);
  color: #fff;
  font-size: 100%;
  padding: 8px 16px;
  height: 40px;
  line-height: 24px;
  font-size: 1rem;
  border-width: 0;
  display: flex;
  flex: none;
  align-items: center;
  margin-left: 1rem;
  cursor: pointer;
}
</style>
