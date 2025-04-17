<template>
  <div class="container">
    <div class="movie-management">
      <h1>🎥 Quản lý Phim</h1>

      <!-- Tabs -->
      <div class="tabs">
        <button @click="selectTab('movies')" :class="{ active: currentTab === 'movies' }">Phim Lẻ</button>
        <button @click="selectTab('series')" :class="{ active: currentTab === 'series' }">Phim Bộ</button>
      </div>

      <!-- Nội dung tab Phim Bộ -->
      <div v-if="currentTab === 'series'" class="tab-content">
        <h2>Quản lý Phim Bộ</h2>
        <button @click="showSeriesMovieForm = true" class="add-button">Thêm Phim Bộ</button>
        <div class="search-bar">
          <input v-model="searchQuery" type="text" placeholder="Tìm kiếm phim bộ" @input="fetchSeries" />
        </div>
        <table class="data-table">
          <thead>
            <tr>
              <th>ID</th>
              <th>Avatar</th>
              <th>Poster</th>
              <th>Tên phim</th>
              <th>Phần</th>
              <th>Diễn viên</th>
              <th>Đạo diễn</th>
              <th>Thể loại</th>
              <th>Quốc gia</th>
              <th>Năm phát hành</th>
              <th>Mô tả</th>
              <th>Phim nóng</th>
              <th>Rating</th>
              <th>Trạng thái</th>
              <th>Hành động</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="paginatedSeries.length === 0 && !loading">
              <td colspan="14">Không có phim bộ nào phù hợp!</td>
            </tr>
            <tr v-for="(series, index) in paginatedSeries" :key="series.seriesId">
              <td>{{ series.seriesId }}</td>
              <td @click="showEpisodes(series.seriesId)" style="cursor: pointer;">
                <img :src="series.avatarUrl" alt="Avatar" width="100" />
              </td>
              <td @click="showEpisodes(series.seriesId)" style="cursor: pointer;">
                <img :src="series.posterUrl" alt="Poster" width="100" />
              </td>
              <td @click="showEpisodes(series.seriesId)" style="cursor: pointer;">
                {{ series.title }}
              </td>
              <td>{{ series.season }}</td>
              <td>
                <div v-for="actor in series.actors" :key="actor.actorId">
                  {{ actor.nameAct }}
                </div>
              </td>
              <td>
                <div v-for="director in series.directors" :key="director.directorID">
                  {{ director.nameDir }}
                </div>
              </td>
              <td>
                <div v-for="category in series.categories" :key="category.categoryId">
                  {{ category.categoryName }}
                </div>
              </td>
              <td>{{ series.nation }}</td>
              <td>{{ series.yearReleased }}</td>
              <td>{{ series.description }}</td>
              <td>{{ series.isHot ? "Có" : "Không" }}</td>
              <td>{{ series.rating }}</td>
              <td>{{ series.status === 1 ? "Đang hoạt động" : "Không hoạt động" }}</td>
              <td>
                <button @click="editSeries(index)" class="edit-button">Sửa</button>
                <button @click="deleteSeries(index)" class="delete-button">Xóa</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div v-if="currentSeriesId" class="episode-list">
      <h2>Danh sách Tập Phim</h2>
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Tập</th>
            <th>Liên kết</th>
            <th>Hành động</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="episode in episodeList" :key="episode.episodeId">
            <td>{{ episode.episodeId }}</td>
            <td>{{ episode.title }}</td>
            <td>{{ episode.linkFilmUrl }}</td>
            <td>
              <button @click="editEpisode(episode.episodeId)" class="edit-button">Sửa</button>
              <button @click="deleteEpisode(episode.episodeId)" class="delete-button">Xóa</button>
            </td>
          </tr>
        </tbody>
      </table>
      <button @click="addEpisode" class="add-button">Thêm Tập Phim</button>
    </div>
    <!-- Nút điều hướng phân trang -->
    <div class="pagination">
      <button v-if="currentTab === 'series'" @click="changePage(page - 1)" :disabled="page === 1">Trước</button>
      <span v-if="currentTab === 'series'">Trang {{ page }} / {{ totalSeriesPages }}</span>
      <button v-if="currentTab === 'series'" @click="changePage(page + 1)"
        :disabled="page === totalSeriesPages">Sau</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Multiselect from "vue-multiselect";
import "vue-multiselect/dist/vue-multiselect.min.css";

export default {
  components: {
    Multiselect,
  },
  data() {
    return {
      currentTab: "movies", // Tab mặc định là "Phim Lẻ"
      paginatedSeries: [], // Dữ liệu phim bộ
      showSeriesMovieForm: false,
      actorOptions: [], // Danh sách diễn viên từ API
      categorieOptions: [],
      directorOptions: [],
      searchQuery: "",
      seriesList: [],
      episodeList: [],
      currentSeriesId: null, // ID của series hiện tại
      loading: false,
      error: null,
      page: 1, // Trang hiện tại
      itemsPerPage: 5, // 5 phim trên mỗi trang
    };
  },
  computed: {
    totalSeriesPages() {
      return Math.ceil(this.seriesList.length / this.itemsPerPage);
    },
  },
  methods: {
    async fetchActorOptions() {
      try {
        const response = await axios.get("http://localhost:5289/api/AdminActor", {
          params: {
            search: "",
            sortBy: "ActorId",
            sortDirection: "asc",
            page: 1,
            pageSize: 50,
          },
        });
        this.actorOptions = response.data.map((actor) => ({
          id: actor.actorId,
          name: actor.nameAct,
        }));
      } catch (error) {
        console.error("Lỗi khi tải danh sách diễn viên:", error);
      }
    },

    async fetchCategorieOptions() {
      try {
        const response = await axios.get("http://localhost:5289/api/AdminCategories", {
          params: {
            search: "",
            sortBy: "CategoryId",
            sortDirection: "asc",
            page: 1,
            pageSize: 50,
          },
        });
        this.categorieOptions = response.data.map((categorie) => ({
          id: categorie.categoryId,
          name: categorie.categoryName,
        }));
      } catch (error) {
        console.error("Lỗi khi tải danh sách thể loại:", error);
      }
    },

    async fetchDirectorOptions() {
      try {
        const response = await axios.get("http://localhost:5289/api/AdminDirectors/List-Directors", {
          params: {
            search: "",
            sortBy: "DirectorId",
            sortDirection: "asc",
            page: 1,
            pageSize: 50,
          },
        });
        this.directorOptions = response.data.map((director) => ({
          id: director.directorID,
          name: director.nameDir,
        }));
      } catch (error) {
        console.error("Lỗi khi tải danh sách đạo diễn:", error);
      }
    },

    async fetchSeries() {
      this.loading = true;
      this.error = null;
      try {
        const response = await axios.get("http://localhost:5289/api/AdminSeries", {
          params: {
            sortBy: "Title",
            search: this.searchQuery.trim()
          },
        });
        this.seriesList = response.data.sort((a, b) => a.seriesId - b.seriesId);
        this.updatePagination();
      } catch (error) {
        console.error("Lỗi khi tải danh sách phim:", error);
        this.error = "Không thể tải danh sách phim!";
      } finally {
        this.loading = false;
      }
    },

    updatePagination() {
      const startIndex = (this.page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      this.paginatedSeries = this.seriesList.slice(startIndex, endIndex);
    },

    async showEpisodes(seriesId) {
      this.currentSeriesId = seriesId; // Lưu seriesId hiện tại
      try {
        const response = await axios.get(`http://localhost:5289/api/AdminEpisode/BySeries/${seriesId}`, {
          params: {
            pageNumber: 1,
            pageSize: 5
          }
        });
        this.episodeList = response.data; // Lưu danh sách tập phim
      } catch (error) {
        console.error("Lỗi khi tải danh sách tập phim:", error);
      }
    },
    toggleEpisodes(seriesId) {
      if (this.activeSeriesId === seriesId) {
        this.activeSeriesId = null; // Đóng danh sách nếu đã mở
      } else {
        this.activeSeriesId = seriesId; // Mở danh sách phim
        this.showEpisodes(seriesId); // Tải danh sách tập phim
      }
    },
    editEpisode(episodeId) {
      // Logic để chỉnh sửa tập phim
      console.log("Chỉnh sửa tập phim với ID:", episodeId);
    },

    deleteEpisode(episodeId) {
      // Logic để xóa tập phim
      console.log("Xóa tập phim với ID:", episodeId);
    },
    addEpisode() {
      // Logic để thêm tập phim
      console.log("Thêm tập phim cho series ID:", this.currentSeriesId);
    },

    changePage(newPage) {
      if (newPage >= 1 && newPage <= Math.ceil(this.seriesList.length / this.itemsPerPage)) {
        this.page = newPage;
        this.updatePagination();
      }
    },

    selectTab(tab) {
      this.currentTab = tab; // Chuyển đổi tab
      this.page = 1; // Đặt lại về trang đầu tiên
      if (tab === "series") {
        this.fetchSeries(); // Tải phim bộ nếu tab là series
      }
    },
  },
  mounted() {
    this.fetchActorOptions();
    this.fetchCategorieOptions();
    this.fetchDirectorOptions();
  },
};
</script>
<style scoped>
@import "/src/assets/css/admin.css";

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
