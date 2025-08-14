<script setup>
// 從 vue 引入 ref, computed, 和 onMounted
import { ref, computed, onMounted } from 'vue';

// 引入圖片資源，這樣才能在 <script> 中當作變數使用
import heartBlack from '@/assets/heartBlack.png';
import heartRed from '@/assets/heartRed.png';

// 引入 JSON 資料和子組件
import albumData from '@/assets/albums.json';
import FavList from '@/components/FavList.vue';

// --- 狀態定義 (State) ---
const albums = ref(albumData.albums);
const search = ref('');
const STORAGE_KEY = 'my-favorite-list';

// 建立一個響應式的收藏列表 ref，用來即時驅動 UI 更新
const favorites = ref([]);

// --- 生命週期 (Lifecycle) ---
// 在組件掛載時，從 localStorage 讀取資料，初始化 favorites 列表
onMounted(() => {
  favorites.value = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');
});

// --- 方法 (Methods) ---

// 建立一個檢查函式，供 template 使用
const isFavorite = (albumId) => {
  // 檢查傳入的 albumId 是否存在於響應式的 favorites 陣列中
  return favorites.value.some(item => item.id === albumId);
};

// 將原本的 addFav 升級為 toggleFavorite
const toggleFavorite = (albumToToggle) => {
  if (isFavorite(albumToToggle.id)) {
    // 如果已經是收藏，就執行「移除」
    favorites.value = favorites.value.filter(item => item.id !== albumToToggle.id);
    alert(`已將「${albumToToggle.name}」移出收藏！`);
  } else {
    // 如果不是收藏，就執行「新增」
    favorites.value.push(albumToToggle);
    alert(`已將「${albumToToggle.name}」加入收藏！`);
  }
  
  // 無論新增或移除，都將最新的 favorites 陣列存回 localStorage
  localStorage.setItem(STORAGE_KEY, JSON.stringify(favorites.value));
  
  // 同時，發送事件通知 FavList 組件也更新
  window.dispatchEvent(new CustomEvent('favorites-updated'));
};

// --- 其他既有邏輯 ---
const asideToggle = ref(false);
const toggleAside = () => {
  asideToggle.value = !asideToggle.value;
};

const filteredAlbums = computed(() => {
  if (!search.value.trim()) {
    return albums.value;
  }
  const searchTerm = search.value.toLowerCase();
  return albums.value.filter(album =>
    album.name.toLowerCase().includes(searchTerm) ||
    album.artists.toLowerCase().includes(searchTerm)
  );
});
</script>

<template>
  <header>
    <div>
      <input type="search" v-model="search" placeholder="搜尋專輯或歌手..." />
      <button @click="toggleAside">
        <img :src="heartRed" alt="收藏列表" />
      </button>
    </div>
  </header>

  <main>
    <div class="card" v-for="album in filteredAlbums" :key="album.id">
      <img :src="album.images" :alt="album.name" />
      <div class="card_body">
        <h6>{{ album.name }}</h6>
        <p>{{ album.artists }}</p>
      </div>
      <div class="card_footer">
        <button class="favoriteBtn" @click="toggleFavorite(album)">
          <img :src="isFavorite(album.id) ? heartRed : heartBlack" alt="收藏專輯" />
        </button>
      </div>
    </div>
  </main>

  <aside :class="{ open: asideToggle }">
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
    }
  }
}
</style>