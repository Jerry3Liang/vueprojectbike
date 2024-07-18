<template>
  <div class="input-group input-group-lg" style="width: 300px; margin-top: 50px; float: right">
    <span class="input-group-text" id="inputGroup-sizing-lg" placeholder="查詢站點地址"
      >查詢站點地址</span
    >
    <input
      type="text"
      class="form-control"
      aria-label="Sizing example input"
      aria-describedby="inputGroup-sizing-lg"
      v-model="keyWord"
    />
    <!-- <button class="btn btn-outline-success" type="button" @click="findKeyWordData(search)">
      搜尋
    </button> -->
  </div>
  <div style="margin:">
    <table class="table table-striped">
      <thead>
        <tr>
          <th scope="col">站點編號</th>
          <th scope="col">站點名稱</th>
          <th scope="col">站點所在區域</th>
          <th scope="col">站點地址</th>
          <th scope="col">
            <span>總車位數量</span>
            <span>
              <button
                type="button"
                class="btn btn-outline-info btn-sm"
                @click="sortTotalBysmallToLarge()"
              >
                ↑
              </button>
              <button
                type="button"
                class="btn btn-outline-info btn-sm"
                @click="sortTotalByLargeToSmall()"
              >
                ↓
              </button>
            </span>
          </th>
          <th scope="col">
            <span>可租借的腳踏車數量</span>
            <span>
              <button
                type="button"
                class="btn btn-outline-info btn-sm"
                @click="sortAvailableRentBysmallToLarge()"
              >
                ↑
              </button>
              <button
                type="button"
                class="btn btn-outline-info btn-sm"
                @click="sortAvailableRentByLargeToSmall()"
              >
                ↓
              </button>
            </span>
          </th>
          <th scope="col">站點緯度</th>
          <th scope="col">站點經度</th>
          <th scope="col">可歸還的腳踏車數量</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="bike in bikes" :key="bike.sno">
          <th scope="row">{{ bike.sno }}</th>
          <td>{{ bike.sna }}</td>
          <td>{{ bike.sarea }}</td>
          <td>{{ bike.ar }}</td>
          <td>{{ bike.total }}</td>
          <td>{{ bike.available_rent_bikes }}</td>
          <td>{{ bike.latitude }}</td>
          <td>{{ bike.longitude }}</td>
          <td>{{ bike.available_return_bikes }}</td>
        </tr>
        <tr></tr>
      </tbody>
    </table>
    <div class="d-flex justify-content-center align-items-center" style="margin-top: 40px">
      <div class="col-4">
        <Paginate
          :first-last-button="true"
          first-button-text="&lt;&lt;"
          last-button-text="&gt;&gt;"
          prev-text="&lt;"
          next-text="&gt;"
          :page-count="pages"
          :initial-page="current"
          v-model="current"
          :click-handler="callFindAllBikeStoreList"
        ></Paginate>
      </div>
    </div>
  </div>
</template>

<script setup>
import axiosApi from 'axios';
import { ref, onMounted, watch } from 'vue';
import Paginate from 'vuejs-paginate-next';

//來自 API 的資料
const datas = ref(null);

//分頁
const pages = ref(0);
const totalbike = ref(0);
const current = ref(1);
const start = ref(1);
const rows = ref(25);
const lastPageRows = ref(0);

//畫面渲染資料
const bikes = ref(null);

//模糊查詢
const keyWord = ref('');

onMounted(function () {
  axiosApi
    .get('https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json')
    .then(function (response) {
      //從 API 獲得的資料賦值給 datas 變數
      datas.value = response.data;

      //從 API 獲得的資料賦值給 bikes 變數
      bikes.value = response.data;
    })
    .catch(function (error) {
      console.log(error);
    });
  callFindAllBikeStoreList(1);
});

function callFindAllBikeStoreList(page) {
  console.log('call callFindAllBikeStoreList', page);

  if (page) {
    start.value = (page - 1) * rows.value;
    current.value = page;
  } else {
    start.value = 0;
    current.value = 1;
  }

  bikes.value = datas.value.slice(start.value, rows.value + 1);

  //分頁
  totalbike.value = datas.value.data.length;
  pages.value = Math.ceil(datas.value.length / rows.value);
  lastPageRows.value = datas.value.data.length % rows.value;
}

watch(
  //要監聽的物件
  () => keyWord.value,
  //監聽到後要實作的方法
  () => {
    console.log(keyWord.value);
    bikes.value = datas.value.filter((data) => data.ar.includes(keyWord.value));
  }
);

function sortTotalBysmallToLarge() {
  bikes.value = datas.value.sort(function (a, b) {
    return a.total - b.total;
  });
}

function sortTotalByLargeToSmall() {
  bikes.value = datas.value.sort(function (a, b) {
    return b.total - a.total;
  });
}

function sortAvailableRentBysmallToLarge() {
  bikes.value = datas.value.sort(function (a, b) {
    return a.available_rent_bikes - b.available_rent_bikes;
  });
}

function sortAvailableRentByLargeToSmall() {
  bikes.value = datas.value.sort(function (a, b) {
    return b.available_rent_bikes - a.available_rent_bikes;
  });
}
</script>

<style scoped></style>
