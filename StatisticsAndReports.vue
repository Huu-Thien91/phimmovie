<template>
  <div>
    <!-- Danh sách phim bộ -->
    <table class="table-auto w-full text-left border">
      <thead>
        <tr class="bg-gray-200">
          <th class="p-2">Poster</th>
          <th class="p-2">Tiêu đề</th>
          <th class="p-2">Thao tác</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="(series, index) in paginatedSeries" :key="series.seriesId">
          <tr class="border-t">
            <td class="p-2 cursor-pointer" @click="toggleEpisodes(series.seriesId)">
              <img :src="series.posterUrl" alt="poster" class="w-20 h-28 object-cover rounded" />
            </td>
            <td class="p-2 cursor-pointer" @click="toggleEpisodes(series.seriesId)">
              {{ series.title }}
            </td>
            <td class="p-2">
              <div class="flex space-x-2 justify-center">
                <button class="bg-blue-500 text-white px-2 py-1 rounded" @click="editSeries(index)">Sửa</button>
                <button class="bg-red-500 text-white px-2 py-1 rounded" @click="deleteSeries(index)">Xoá</button>
                <button class="bg-green-500 text-white px-2 py-1 rounded" @click="addEpisode(series.seriesId)">Thêm tập</button>
              </div>
            </td>
          </tr>

          <!-- Hiển thị danh sách tập phim khi bấm vào series -->
          <tr v-if="selectedSeriesId === series.seriesId">
            <td colspan="3" class="p-4 bg-gray-50">
              <div class="flex justify-between items-center mb-2">
                <h3 class="font-semibold text-lg">Danh sách tập phim</h3>
              </div>
              <!-- Hiển thị 5 tập phim đầu tiên -->
              <div v-for="(ep, epIndex) in episodes.slice(0, 5)" :key="ep.episodeId">
                <div class="flex justify-between items-center p-2 border-t">
                  <span>{{ ep.title }}</span>
                  <div class="flex space-x-2">
                    <button class="bg-yellow-400 text-white px-2 py-1 rounded" @click="editEpisode(ep)">Sửa</button>
                    <button class="bg-red-400 text-white px-2 py-1 rounded" @click="deleteEpisode(ep.episodeId)">Xoá</button>
                  </div>
                </div>
              </div>

              <!-- Tải thêm tập phim khi cuộn -->
              <div v-if="loadingEpisodes" class="text-center p-4">Đang tải...</div>
              <div v-if="allEpisodesLoaded" class="text-center p-4 text-gray-500">Tất cả tập phim đã được tải</div>

              <!-- Nút cuộn tải thêm -->
              <button v-if="!allEpisodesLoaded" @click="loadMoreEpisodes" class="bg-blue-500 text-white px-4 py-2 rounded-full mt-4 block mx-auto">
                Tải thêm tập
              </button>
            </td>
          </tr>
        </template>
      </tbody>
    </table>

    <!-- Phân trang phim bộ -->
    <div class="flex justify-center mt-4">
      <button class="px-3 py-1 mx-1 bg-gray-300 rounded" @click="changePage(page - 1)" :disabled="page <= 1">&lt;</button>
      <span class="px-3 py-1">Trang {{ page }} / {{ totalSeriesPages }}</span>
      <button class="px-3 py-1 mx-1 bg-gray-300 rounded" @click="changePage(page + 1)" :disabled="page >= totalSeriesPages">&gt;</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      seriesList: [],
      page: 1,
      pageSize: 5, // Hiển thị 5 bộ phim mỗi trang
      totalSeriesPages: 1,
      selectedSeriesId: null,
      episodes: [],
      loading: false,
      loadingEpisodes: false,
      allEpisodesLoaded: false,
      episodePage: 1,
      episodePageSize: 5, // Mỗi lần tải 5 tập phim
    };
  },
  computed: {
    paginatedSeries() {
      const filtered = this.seriesList;
      this.totalSeriesPages = Math.ceil(filtered.length / this.pageSize);
      return filtered.slice((this.page - 1) * this.pageSize, this.page * this.pageSize);
    }
  },
  methods: {
    async fetchSeries() {
      try {
        this.loading = true;
        const response = await axios.get(`http://localhost:5148/api/AdminSeries?sortBy=Title&sortDirection=asc&page=${this.page}&pageSize=${this.pageSize}`);
        this.seriesList = response.data;
        this.loading = false;
      } catch (error) {
        console.error('Lỗi khi tải danh sách phim bộ:', error);
        this.loading = false;
      }
    },
    changePage(newPage) {
      if (newPage >= 1 && newPage <= this.totalSeriesPages) {
        this.page = newPage;
        this.fetchSeries();
      }
    },
    toggleEpisodes(seriesId) {
      if (this.selectedSeriesId === seriesId) {
        this.selectedSeriesId = null;
        this.episodes = [];
        this.allEpisodesLoaded = false;
      } else {
        this.selectedSeriesId = seriesId;
        this.fetchEpisodes(seriesId);
      }
    },
    async fetchEpisodes(seriesId) {
      try {
        this.loadingEpisodes = true;
        const res = await axios.get(`/api/AdminEpisode/BySeries/${seriesId}?pageNumber=${this.episodePage}&pageSize=${this.episodePageSize}`);
        if (res.data.length < this.episodePageSize) {
          this.allEpisodesLoaded = true;
        }
        this.episodes = [...this.episodes, ...res.data];
        this.loadingEpisodes = false;
      } catch (err) {
        console.error('Lỗi khi tải tập phim:', err);
        this.loadingEpisodes = false;
      }
    },
    loadMoreEpisodes() {
      if (!this.allEpisodesLoaded && !this.loadingEpisodes) {
        this.episodePage++;
        this.fetchEpisodes(this.selectedSeriesId);
      }
    },
    editSeries(index) {
      // Logic để mở form sửa phim bộ (nếu cần làm thêm modal)
    },
    async deleteSeries(index) {
      const id = this.seriesList[index].seriesId;
      try {
        await axios.delete(`/api/AdminSeries/${id}`);
        this.fetchSeries();
      } catch (err) {
        console.error('Lỗi xoá phim bộ:', err);
      }
    },
    editEpisode(ep) {
      // Logic để sửa tập phim
    },
    async deleteEpisode(episodeId) {
      try {
        await axios.delete(`/api/episodes/${episodeId}`);
        this.fetchEpisodes(this.selectedSeriesId);
      } catch (err) {
        console.error('Lỗi xoá tập:', err);
      }
    },
    addEpisode(seriesId) {
      // Logic để mở modal thêm tập phim
    }
  },
  created() {
    this.fetchSeries();
  },
  watch: {
    selectedSeriesId(newVal) {
      if (newVal) {
        this.episodes = [];
        this.episodePage = 1;
        this.allEpisodesLoaded = false;
        this.fetchEpisodes(newVal);
      }
    }
  }
};
</script>

<style scoped>
/* Bảng phim bộ */
.table-auto {
  width: 100%;
  border-collapse: collapse;
}
.table-auto th,
.table-auto td {
  padding: 12px;
  border: 1px solid #ddd;
}

.bg-gray-200 {
  background-color: #f7f7f7;
}
.bg-gray-50 {
  background-color: #f9fafb;
}

.cursor-pointer {
  cursor: pointer;
}

.text-left {
  text-align: left;
}

.text-center {
  text-align: center;
}

.p-2 {
  padding: 0.5rem;
}

.p-4 {
  padding: 1rem;
}

.mt-4 {
  margin-top: 1rem;
}

.mr-4 {
  margin-right: 1rem;
}

/* Button styles */
.bg-blue-500 {
  background-color: #3b82f6;
}
.bg-yellow-400 {
  background-color: #facc15;
}
.bg-red-500 {
  background-color: #ef4444;
}
.bg-green-500 {
  background-color: #10b981;
}

.text-white {
  color: white;
}

.rounded {
  border-radius: 0.375rem;
}

.px-2 {
  padding-left: 0.5rem;
  padding-right: 0.5rem;
}

.py-1 {
  padding-top: 0.25rem;
  padding-bottom: 0.25rem;
}

.px-4 {
  padding-left: 1rem;
  padding-right: 1rem;
}

.py-2 {
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
}

.rounded-full {
  border-radius: 9999px;
}

.mt-4 {
  margin-top: 1rem;
}

.block {
  display: block;
}

.mx-auto {
  margin-left: auto;
  margin-right: auto;
}

/* Hover Effects */
button:hover {
  opacity: 0.9;
}
/* ok */
/* Form Overlay */
.form-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.form-group {
  flex: 1;
}

.form-container {
  background: #fff;
  padding: 30px;
  border-radius: 10px;
  width: 600px;
  max-width: 90%;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

.filter-button {
  padding: 10px 15px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.filter-button:hover {
  background-color: #2980b9;
}


.form-group {
  margin-bottom: 15px;
}

.filters {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
}

.movie-form {
  display: flex;
  flex-direction: column;
  /* Stack columns vertically */
}

.form-columns {
  display: flex;
  justify-content: space-between;
}

.form-column {
  width: 48%;
}

.left-column {
  margin-right: 20px;
  /* Space between columns */
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #333;
}

.form-group input,
.form-group select {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.form-group input:focus,
.form-group select:focus {
  border-color: #4caf50;
  outline: none;
}


h2 {
  margin-bottom: 20px;
  font-size: 1.5rem;
  text-align: center;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.button-group {
  display: flex;
  justify-content: space-between;
  /* Space buttons apart */
  margin-top: 20px;
}

.submit-button {
  width: 48%;
  background: #4caf50;
  color: #fff;
  border: none;
  padding: 12px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}


.cancel-button {
  width: 48%;
  background: #f44336;
  color: #fff;
  border: none;
  padding: 12px;
  border-radius: 5px;
  cursor: pointer;
}

.submit-button:hover,
.cancel-button:hover {
  opacity: 0.9;
}

.movie-management {
  background-color: #fff;
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  animation: fadeIn 1s ease-in-out;
  width: 120%;
}

.tabs {
  display: flex;
  gap: 20px;
}

.tabs button,
.add-button,
.add-button1 {
  padding: 12px 50px;
  font-size: 18px;
  background-color: #f0f0f0;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.tabs button.active,
.add-button,
.add-button1 {
  background-color: #4caf50;
  color: white;
}

.add-button1 {
  margin-left: 10px;
}

.tab-content {
  margin-top: 30px;
}

.search-bar {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 20px;
}

.search-bar input {
  width: 300px;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ddd;
  border-radius: 6px;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.data-table th,
.data-table td {
  padding: 12px;
  border: 1px solid #ddd;
  text-align: center;
}

.data-table th {
  background-color: #f4f4f4;
}

.edit-button,
.delete-button {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 6px;
  cursor: pointer;
  margin-right: 5px;
  margin-top: 5px;
}

.add-button,
.add-button1 {
  background-color: rgb(12, 187, 245);
}

.delete-button {
  background-color: #f44336;
}

.edit-button:hover {
  background-color: #45a049;
}

.delete-button:hover {
  background-color: #e53935;
}

.movie-form {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-top: 20px;
}

.movie-form .form-group {
  display: flex;
  flex-direction: column;
}

.movie-form .form-group label {
  margin-bottom: 5px;
  font-weight: bold;
}

.movie-form .form-group input,
.movie-form .form-group select {
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ddd;
  border-radius: 6px;
}

.submit-button {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 12px 20px;
  font-size: 16px;
  border-radius: 6px;
  cursor: pointer;
  align-self: flex-start;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.submit-button:hover {
  background-color: #45a049;
}

.movie-list,
.series-list {
  margin-bottom: 20px;
}

.movie-list ul,
.series-list ul {
  list-style-type: none;
  padding: 0;
}

.movie-list li,
.series-list li {
  padding: 10px;
  background-color: #f0f0f0;
  margin-bottom: 5px;
  border-radius: 6px;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
}

.pagination button {
  padding: 8px 12px;
  border: none;
  background-color: #3498db;
  color: white;
  cursor: pointer;
}

.pagination button:disabled {
  background-color: #bdc3c7;
  cursor: not-allowed;
}

.pagination span {
  font-weight: bold;
}

.link-film-url {
  width: 100px !important;
  height: auto !important;
}
</style>
