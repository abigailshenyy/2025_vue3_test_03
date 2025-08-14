<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

// 確保 STORAGE_KEY 與 App.vue 中的完全一致
const STORAGE_KEY = 'my-favorite-list';

const favList = ref([]);

// 建立一個專門用來從 localStorage 讀取並更新列表的函式
const loadFavorites = () => {
  const storedData = localStorage.getItem(STORAGE_KEY);
  favList.value = storedData ? JSON.parse(storedData) : [];
};

// 移除收藏列表的函式
const removeFav = (target) => {
  // 從當前列表中移除
  favList.value = favList.value.filter(item => item.id !== target.id);
  // 將更新後的列表存回 localStorage
  localStorage.setItem(STORAGE_KEY, JSON.stringify(favList.value));
  // 同樣發送事件，確保 App.vue 中的愛心也能同步更新
  window.dispatchEvent(new CustomEvent('favorites-updated'));
};

// 在組件掛載時
onMounted(() => {
  // 第一次載入時，先執行一次
  loadFavorites();
  
  // 開始監聽 App.vue 發出的 'favorites-updated' 事件
  window.addEventListener('favorites-updated', loadFavorites);
});

// 在組件被銷毀時，移除事件監聽
onUnmounted(() => {
  window.removeEventListener('favorites-updated', loadFavorites);
});
</script>

<template>
  <div class="favList">
    <div class="nodata" v-if="!favList.length">還沒有加入收藏喔</div>

    <div class="container" v-else>
      <div class="list" v-for="item in favList" :key="item.id">
        <img :src="item.images" />
        <div class="list_content">
          <h6>{{ item.name }}</h6>
          <p>{{ item.artists }}</p>
        </div>
        <div class="list_action">
          <button @click="removeFav(item)">X</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.favList {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  gap: 0.5rem;
  .nodata {
    width: 100%;
    height: 10rem;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .container {
    overflow: hidden;
    padding: 5rem 0.5rem 0.5rem 0.5rem;
  }
  .list {
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 0.5rem;
    margin-bottom: 0.5rem;
    img {
      width: 4rem;
    }
    &_content {
      flex: 0 1 calc(100% - 7.5rem);
      h6 {
        font-size: 0.8rem;
      }
      p {
        font-size: 0.75rem;
      }
    }
    &_action {
      display: flex;
      flex-direction: row;
      align-items: center;
      width: 2.5rem;
      button {
        background-color: transparent;
        color: rgb(175, 50, 119);
      }
    }
  }
}
</style>