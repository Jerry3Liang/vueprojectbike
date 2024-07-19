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
      <tbody v-if="paginatedData">
        <tr v-for="bike in paginatedData" :key="bike">
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

    <ul class="pagination justify-content-center">
      <li class="page-item disabled">
        <button class="page-link" @click="backPage">Previous</button>
      </li>

      <li class="page-item">
        <button
          class="page-link"
          v-for="item in pages"
          :key="item.index"
          @click="() => goToPage(item)"
        >
          {{ item }}
        </button>
      </li>

      <li class="page-item">
        <button class="page-link" @click="nextPage">Next</button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import axiosApi from 'axios';
import { ref, onMounted, watch, computed } from 'vue';

//動態獲取資料
async function getData() {
  try {
    const response = await axiosApi.get(
      'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
    );
    return response;
  } catch (error) {
    console.log(error);
  }
}

// //來自 API 的資料
const datas = ref(null);
getData().then((res) => {
  datas.value = res.data;
});

console.log(datas);

//從 API 獲得的資料賦值給 bikes 變數
const bikes = ref(null);
getData().then((res) => {
  bikes.value = res.data;
});
console.log(bikes);

//分頁
//總頁數
const pages = ref(0);
const totalbike = ref(0);

//當前頁數 (預設為第 1頁)
const current = ref(1);

const start = ref(1);

// 一頁要幾筆資料
const rows = ref(25);

//每 25筆原始資料成為一個 paginatedData(動態)
const paginatedData = ref(
  computed(() => {
    if (bikes.value) {
      return bikes.value.slice((current.value - 1) * rows.value, current.value * rows.value + 1);
    }
    return bikes.value;
  })
);

//下一頁
const nextPage = () => {
  if (current.value !== Math.ceil(bikes.value.length / rows.value)) {
    current.value += 1;
  }
};

// 上一頁
const backPage = () => {
  if (current.value !== 1) {
    current.value -= 1;
  }
};

//前往某一頁
const goToPage = (numPage) => {
  current.value = numPage;
};

const lastPageRows = ref(0);

//模糊查詢的關鍵字
const keyWord = ref('');

// onMounted(function () {
//   axiosApi
//     .get('https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json')
//     .then(function (response) {
//       totalbike.value = response.data.length;

//       pages.value = Math.ceil(response.data.length / rows.value);
//     })
//     .catch(function (error) {
//       console.log(error);
//     });
// });

// function callFindAllBikeStoreList(page) {
//   console.log('call callFindAllBikeStoreList', page);

//   if (page) {
//     start.value = (page - 1) * rows.value;
//     current.value = page;
//   } else {
//     start.value = 0;
//     current.value = 1;
//   }

//   bikes.value = datas.value.slice(start.value, rows.value + 1);

//   //分頁

//   pages.value = Math.ceil(datas.value.length / rows.value);
//   lastPageRows.value = datas.value.length % rows.value;
// }

//透過關鍵字搜尋
watch(
  //要監聽的物件
  () => keyWord.value,
  //監聽到後要實作的方法
  () => {
    console.log(keyWord.value);
    bikes.value = datas.value.filter((data) => data.ar.includes(keyWord.value));
  }
);

//根據 total 數量排序 (小到大)
function sortTotalBysmallToLarge() {
  bikes.value = datas.value.sort(function (a, b) {
    return a.total - b.total;
  });
}

//根據 total 數量排序 (大到小)
function sortTotalByLargeToSmall() {
  bikes.value = datas.value.sort(function (a, b) {
    return b.total - a.total;
  });
}

//根據 available_rent_bikes 數量排序 (小到大)
function sortAvailableRentBysmallToLarge() {
  bikes.value = datas.value.sort(function (a, b) {
    return a.available_rent_bikes - b.available_rent_bikes;
  });
}

//根據 available_rent_bikes 數量排序 (大到小)
function sortAvailableRentByLargeToSmall() {
  bikes.value = datas.value.sort(function (a, b) {
    return b.available_rent_bikes - a.available_rent_bikes;
  });
}
</script>

<style scoped></style>
