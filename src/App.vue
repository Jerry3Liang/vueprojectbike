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
      <li class="page-item">
        <button class="page-link" @click="firstPage" v-if="current !== 1">第一頁</button>
        <button class="page-link disabled" @click="firstPage" v-if="current === 1">第一頁</button>
      </li>
      <li class="page-item">
        <button class="page-link" @click="backPage" v-if="current !== 1">上一頁</button>
        <button class="page-link disabled" @click="backPage" v-if="current === 1">上一頁</button>
      </li>

      <li
        class="page-item"
        v-for="(page, index) in pages"
        :key="index"
        @click="() => goToPage(page)"
        v-show="index >= start - 1 && index < end"
      >
        <button class="page-link" v-if="page !== current">
          {{ page }}
        </button>
        <button class="page-link active" v-if="page === current">
          {{ page }}
        </button>
      </li>

      <li class="page-item">
        <button class="page-link" @click="nextPage" v-if="current !== pages">下一頁</button>
        <button class="page-link disabled" @click="nextPage" v-if="current === pages">
          下一頁
        </button>
      </li>
      <li class="page-item">
        <button class="page-link" @click="lastPage" v-if="current !== pages">最後一頁</button>
        <button class="page-link disabled" @click="lastPage" v-if="current === pages">
          最後一頁
        </button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import axiosApi from 'axios';
import { ref, onMounted, watch, computed } from 'vue';

const bikeInfo = ref([]);

// 取得YouBike資料
const getBikeInfo = async () => {
  const response = await axiosApi.get(
    'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
  );

  bikeInfo.value = response.data;
  console.log(bikeInfo.value);
};

onMounted(async () => {
  await getBikeInfo();
});

// 關鍵字搜尋
const searchAr = ref('');
const bikeinfoFilterBySearchAr = computed(() => {
  return bikeInfo.value.filter((item) => item.ar.includes(searchAr.value));
});

//透過關鍵字模糊查詢
const keyWord = ref('');
watch(
  //要監聽的物件
  () => keyWord.value,
  //監聽到後要實作的方法
  () => {
    current.value = 1;
    bikes.value = datas.value.filter((data) => data.ar.includes(keyWord.value));
  }
);

//關鍵字 Highlight
const heightligthAr = (ar) => {
  if (searchAr.value) {
    return ar.replace(
      searchAr.value,
      `<span class="text-red-500 font-bold">${searchAr.value}</span>`
    );
  } else {
    return ar;
  }
};

//動態獲取資料
async function getData() {
  try {
    const response = await axiosApi.get(
      'https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json'
    );

    //來自 API 的資料
    datas.value = response.data;

    //從 API 獲得的資料賦值給 bikes 變數
    bikes.value = response.data;

    //所有 data 數
    totalbike.value = response.data.length;

    pages.value = Math.ceil(totalbike.value / rows.value);
    return response;
  } catch (error) {
    console.log(error);
  }
}

//呼叫動態獲取資料
getData();

//來自 API 的資料
const datas = ref(null);
console.log(datas);

//bikes 變數 (渲染畫面用)
const bikes = ref(null);
console.log(bikes);

//分頁
//data 總數
const totalbike = ref(0);
console.log(totalbike);

// 一頁要幾筆資料
const rows = ref(25);

//當前頁數 (預設為第 1頁)
const current = ref(1);

const pages = ref(0);

//每 25筆原始資料成為一個 paginatedData(動態)
const paginatedData = computed(() => {
  if (bikes.value) {
    return bikes.value.slice((current.value - 1) * rows.value, current.value * rows.value);
  }
  return bikes.value;
});

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

//前往第一頁
const firstPage = () => {
  if (current.value !== 1) {
    current.value = 1;
  }
};

//前往最後一頁
const lastPage = () => {
  if (current.value !== Math.ceil(bikes.value.length / rows.value)) {
    current.value = Math.ceil(bikes.value.length / rows.value);
  }
};

const start = computed(() => {
  // 設定條件：如果 current.value + 10 會小於總頁數，那就讓 start = current.value
  if (current.value + 10 <= Math.ceil(bikes.value.length / rows.value)) {
    return current.value;

    // 如果會大於總頁數的話，那就讓 start = 總頁數-9 (從後面往前算10頁)
  } else if (
    current.value + 10 > Math.ceil(bikes.value.length / rows.value) ||
    current.value === Math.ceil(bikes.value.length / rows.value)
  ) {
    return Math.ceil(bikes.value.length / rows.value) - 9;
  }
  return current.value;
});

const end = computed(() => {
  if (current.value + 10 <= Math.ceil(bikes.value.length / rows.value)) {
    return current.value + 9;
  } else if (
    current.value + 10 > Math.ceil(bikes.value.length / rows.value) ||
    current.value === Math.ceil(bikes.value.length / rows.value)
  ) {
    return Math.ceil(bikes.value.length / rows.value);
  }
  return current.value;
});

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
