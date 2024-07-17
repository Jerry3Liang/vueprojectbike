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
    />
  </div>
  <div style="margin:">
    <table class="table table-striped">
      <thead>
        <tr>
          <th scope="col">站點編號</th>
          <th scope="col">站點名稱</th>
          <th scope="col">站點所在區域</th>
          <th scope="col">站點地址</th>
          <th scope="col">總車位數量</th>
          <th scope="col">可租借的腳踏車數量</th>
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
import { ref, onMounted } from 'vue';
import Paginate from 'vuejs-paginate-next';

//分頁
const pages = ref(0);
const totalbike = ref(0);
const current = ref(1);
const start = ref(1);
const rows = ref(20);
const lastPageRows = ref(0);

const bikes = ref(null);

onMounted(function () {
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

  axiosApi
    .get('https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json')
    .then(function (response) {
      bikes.value = response.data;

      //分頁
      totalbike.value = response.data.length;
      pages.value = Math.ceil(response.data.length / rows.value);
      lastPageRows.value = response.data.length % rows.value;

      console.log(response);
    })
    .catch(function (error) {
      console.log(error);
    });
}
</script>

<style scoped></style>
