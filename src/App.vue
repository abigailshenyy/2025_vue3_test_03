<script setup>
// 1. 從 vue 引入 ref 和 computed
import { ref, computed } from 'vue';

// 2. 像引入組件一樣，直接引入 albums.json 檔案
import albumData from '@/assets/albums.json';

// 任務1. 引入FavList組件
import FavList from '@/components/FavList.vue';

// 如果使用pinia...

// 任務2. 顯示專輯資料
// 3. 直接使用 import 進來的 albumData 初始化 albums
const albums = ref(albumData.albums);
const search = ref('');
const STORAGE_KEY = 'my-favorite-list';

// `defaultData` 和 `onMounted` 區塊都已不再需要，已被移除

// 任務3: 開啟關閉側拉選單(收藏列表)
const asideToggle = ref(false);
const toggleAside = () => {
  asideToggle.value = !asideToggle.value;
};

// 任務4. 專輯資料可以被input搜尋
const filteredAlbums = computed(() => {
  // 如果搜尋框為空，返回全部專輯
  if (!search.value.trim()) {
    return albums.value;
  }
  // 進行不分大小寫的搜尋
  const searchTerm = search.value.toLowerCase();
  return albums.value.filter(album =>
    album.name.toLowerCase().includes(searchTerm) ||
    album.artists.toLowerCase().includes(searchTerm)
  );
});

// 任務5. 加入我的收藏
const addFav = (albumToAdd) => {
  // 1. 讀取現有的收藏列表，如果沒有則初始化為空陣列
  const currentFavs = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');

  // 2. 檢查專輯是否已在收藏中，防止重複
  const isAlreadyFav = currentFavs.some(item => item.id === albumToAdd.id);
  if (isAlreadyFav) {
    alert('此專輯已經在您的收藏中了！');
    return;
  }

  // 3. 將新專輯加入陣列
  currentFavs.push(albumToAdd);

  // 4. 將更新後的陣列存回 localStorage
  localStorage.setItem(STORAGE_KEY, JSON.stringify(currentFavs));
  
  alert(`已將「${albumToAdd.name}」加入收藏！`);
  
  // 5. 通知 FavList 組件更新
  window.dispatchEvent(new CustomEvent('favorites-updated'));
};
</script>

<template>
  <header>
    <div>
      <!-- 任務4. -->
      <input type="search" v-model="search" placeholder="搜尋專輯或歌手..." />
      <!-- 任務3. -->
      <button @click="toggleAside">
        <img src="~@/assets/heartRed.png" alt="收藏列表" />
      </button>
    </div>
  </header>

  <main>
    <!-- 任務2 & 4.-->
    <div class="card" v-for="album in filteredAlbums" :key="album.id">
      <img :src="album.images" :alt="album.name" />
      <div class="card_body">
        <h6>{{ album.name }}</h6>
        <p>{{ album.artists }}</p>
      </div>
      <div class="card_footer">
        <!-- 任務5.-->
        <button class="favoriteBtn" @click="addFav(album)">
          <img src="~@/assets/heartBlack.png" alt="收藏專輯" />
        </button>
      </div>
    </div>
  </main>

  <!-- 任務3.-->
  <aside :class="{ open: asideToggle }">
    <!-- 任務1.-->
    <FavList />
  </aside>
</template>

<style lang="scss">
button {
  width: 2rem;
  height: 2rem;
  border-radius: 2rem;
  padding: 0.2rem;
  img {
    width: 100%;
  }
}
</style>

<style lang="scss" scoped>
header {
  position: fixed;
  width: 100%;
  z-index: 2;
  background-color: #f9f9f9;
  >div {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    width: 100%;
    max-width: 57rem;
    margin: auto;
    padding: 0.5rem;
    box-sizing: border-box;

    input[type="search"] {
      padding: 0.5rem;
      border: 1px solid #ccc;
      border-radius: 0.5rem;
      width: 50%;
    }
  }
}
main {
  width: 100%;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: flex-start;
  gap: 1rem;
  max-width: 57rem;
  margin: 3rem auto;
  padding: 0.5rem;
  padding-top: 3.5rem;
  box-sizing: border-box;
}
aside {
  width: 20rem;
  height: 100vh;
  position: fixed;
  overflow: hidden;
  overflow-y: auto;
  z-index: 3;
  top: 0;
  right: -20rem;

  background-color: #2a2a2b;
  color: #f9f9f9;

  box-shadow: -2px 0px 5px rgba(0, 0, 0, 0.3);
  transition: right 0.5s ease-in-out;

  &.open {
    right: 0;
  }
}

.card {
  width: 12rem;
  padding: 0.75rem;
  border-radius: 0.5rem;
  background-color: #1a1a1a;
  color: #f9f9f9;
  display: flex;
  flex-direction: column;

  img {
    width: 100%;
    aspect-ratio: 1 / 1;
    object-fit: cover;
    border-radius: 0.25rem;
  }

  &_body {
    height: 3.5rem;
    overflow: hidden;
    margin-top: 0.5rem;
    h6 {
      font-size: 0.9rem;
      margin: 0;
    }
    p {
      font-size: 0.8rem;
      margin: 0.25rem 0 0 0;
      color: #b3b3b3;
    }
  }
  &_footer {
    margin-top: auto;
    padding-top: 0.5rem;
    text-align: right;
    .favoriteBtn {
      background-color: #f9f9f9;
      filter: sepia(0);
      transition: filter 0.2s;
      &:hover {
        filter: sepia(1);
      }
    }
  }
}
</style>