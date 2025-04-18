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
          <input v-model="searchQuery" type="text" placeholder="Tìm kiếm phim bộ" @input="searchSeries" />
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
              <td colspan="15">Không có phim bộ nào phù hợp!</td>
            </tr>

            <template v-for="(series, index) in paginatedSeries" :key="series.seriesId">
              <tr>
                <td>{{ series.seriesId }}</td>
                <td><img :src="series.avatarUrl" alt="Avatar" width="100" /></td>
                <td><img :src="series.posterUrl" alt="Poster" width="100" /></td>
                <td>{{ series.title }}</td>
                <td>{{ series.season }}</td>
                <td>
                  <div v-for="actor in series.actors" :key="actor.actorId">{{ actor.nameAct }}</div>
                </td>
                <td>
                  <div v-for="director in series.directors" :key="director.directorID">{{ director.nameDir }}</div>
                </td>
                <td>
                  <div v-for="category in series.categories" :key="category.categoryId">{{ category.categoryName }}
                  </div>
                </td>
                <td>{{ series.nation }}</td>
                <td>{{ series.yearReleased }}</td>
                <td>{{ series.description }}</td>
                <td>{{ series.isHot ? 'Có' : 'Không' }}</td>
                <td>{{ series.rating }}</td>
                <td>{{ series.status === 1 ? 'Đang hoạt động' : 'Không hoạt động' }}</td>
                <td>
                  <button @click="toggleEpisodes(series)">
                    {{ series.showEpisodes ? 'Ẩn' : 'Xem' }}
                  </button>
                  <button @click="editSeries(index)" class="edit-button">Sửa</button>
                  <button @click="deleteSeries(index)" class="delete-button">Xóa</button>
                </td>
              </tr>

              <!-- Danh sách tập phim -->
              <tr v-if="series.showEpisodes">
                <td colspan="15">
                  <div class="episodes-list">
                    <h4>Danh sách tập phim</h4>
                    <button @click="openAddEpisodeModal(series.seriesId)" class="add-button">➕ Thêm Tập Phim</button>
                    <table class="data-table">
                      <thead>
                        <tr>
                          <th>Tập</th>
                          <th>Tiêu đề</th>
                          <th>Link</th>
                          <th>Hành động</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="episode in series.episodes" :key="episode.episodeId">
                          <td>Tập {{ episode.episodeNumber }}</td>
                          <td>{{ episode.title }}</td>
                          <td><a :href="episode.linkFilmUrl" target="_blank">Xem</a></td>
                          <td>
                            <button @click="openEditEpisodeModal(episode)" class="edit-button">Sửa</button>
                            <button @click="deleteEpisode(episode.episodeId, series.seriesId)"
                              class="delete-button">Xóa</button>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                </td>
              </tr>
            </template>
          </tbody>
        </table>
      </div>

      <!-- Phân trang -->
      <div class="pagination" v-if="currentTab === 'series'">
        <button @click="changePage(page - 1)" :disabled="page === 1">Trước</button>
        <span>Trang {{ page }} / {{ totalSeriesPages }}</span>
        <button @click="changePage(page + 1)" :disabled="page === totalSeriesPages">Sau</button>
      </div>
    </div>
  </div>

  <!-- Modal Thêm Tập Phim -->
  <div v-if="showAddEpisodeModal" class="modal">
    <div class="modal-content">
      <h3>Thêm Tập Phim</h3>
      <label for="episodeNumber">Số tập:</label>
      <input v-model="newEpisode.episodeNumber" type="number" min="1" id="episodeNumber" required />

      <label for="title">Tiêu đề:</label>
      <input v-model="newEpisode.title" type="text" id="title" required />

      <label for="linkFilmUrl">Link phim:</label>
      <input v-model="newEpisode.linkFilmUrl" type="url" id="linkFilmUrl" required />

      <!-- Container chứa nút Thêm và Hủy -->
      <div class="buttons-container">
        <button @click="addEpisode">Thêm Tập Phim</button>
        <button @click="closeAddEpisodeModal">Hủy</button>
      </div>
    </div>
  </div>

  <!-- Modal Sửa Tập Phim -->
  <div v-if="showEditEpisodeModal" class="modal">
    <div class="modal-content">
      <h3>Sửa Tập Phim</h3>
      <label for="editEpisodeNumber">Số tập:</label>
      <input v-model="editingEpisode.episodeNumber" type="number" min="1" id="editEpisodeNumber" required />

      <label for="editTitle">Tiêu đề:</label>
      <input v-model="editingEpisode.title" type="text" id="editTitle" required />

      <label for="editLinkFilmUrl">Link phim:</label>
      <input v-model="editingEpisode.linkFilmUrl" type="url" id="editLinkFilmUrl" required />

      <div class="buttons-container">
        <button @click="updateEpisode">Cập nhật Tập Phim</button>
        <button @click="closeEditEpisodeModal">Hủy</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { ref } from 'vue';
import Multiselect from "vue-multiselect";
import "vue-multiselect/dist/vue-multiselect.min.css";
const currentTab = ref("movies"); // Tab mặc định là "Phim Lẻ"
const paginatedSeries = ref([]); // Dữ liệu phim bộ
const selectTab = (tab) => {
  currentTab.value = tab; // Chuyển đổi tab
};

export default {
  components: {
    Multiselect,
  },
  data() {
    return {
      showAddEpisodeModal: false,
      showEditEpisodeModal: false,
      editingEpisode: {},
      selectedSeriesId: null,
      showSeriesMovieForm: false,
      showUpdateSeriesMovieForm: false,
      actorOptions: [], // Danh sách diễn viên từ API
      categorieOptions: [],
      directorOptions: '',
      searchQuery: "",
      allMovies: [],
      allSeries: [],
      series: [],
      loading: false,
      error: null,
      avatarPreview: null,
      posterPreview: null,
      currentTab: "movies",
      seriesList: [],
      page: 1, // Trang hiện tại
      itemsPerPage: 5, // 5 phim trên mỗi trang
    };
  },
  computed: {
    // Tổng số trang phim
    totalSeriesPages() {
      return Math.ceil(this.seriesList.length / this.itemsPerPage);
    },
    paginatedSeries() {
      const start = (this.page - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.seriesList.slice(start, end);
    },
    filteredSeries() {
      const query = this.searchQuery.trim().toLowerCase();
      return this.seriesList.filter(
        (series) =>
          series.title.toLowerCase().includes(query) ||
          series.description.toLowerCase().includes(query)
      );
    }
  },
  methods: {
    async toggleEpisodes(series) {
      if (!series.showEpisodes) {
        try {
          // Gọi API để lấy danh sách tập phim
          const response = await axios.get(
            `http://localhost:5289/api/AdminEpisode/BySeries/${series.seriesId}?pageNumber=1&pageSize=100`
          );
          console.log(response.data); // Kiểm tra dữ liệu API trả về

          // Cập nhật tập phim vào series (Cập nhật trực tiếp vào series)
          series.episodes = response.data; // Cập nhật trực tiếp mảng hoặc đối tượng
        } catch (error) {
          console.error("Lỗi khi tải danh sách tập phim:", error);
          series.episodes = []; // Gán mảng rỗng khi có lỗi
        }
      }

      // Đảo trạng thái hiển thị
      series.showEpisodes = !series.showEpisodes; // Đảo ngược giá trị của showEpisodes
    },

    openAddEpisodeModal(seriesId) {
      this.selectedSeriesId = seriesId;
      this.newEpisode = {
        episodeNumber: 1,
        title: '',
        linkFilmUrl: ''
      };
      this.showAddEpisodeModal = true;
    },

    async addEpisode() {
      try {
        const formData = new FormData();
        formData.append("SeriesId", this.selectedSeriesId);
        formData.append("EpisodeNumber", this.newEpisode.episodeNumber);
        formData.append("Title", this.newEpisode.title);
        formData.append("LinkFilmUrl", this.newEpisode.linkFilmUrl);

        const response = await axios.post(
          "http://localhost:5289/api/AdminEpisode/AddEpisode",
          formData,
          { headers: { "Content-Type": "multipart/form-data" } }
        );

        const addedEpisode = response.data;
        const series = this.seriesList.find(s => s.seriesId === this.selectedSeriesId);
        if (series) {
          series.episodes.push(addedEpisode); // Cập nhật danh sách tập phim
        }

        this.showAddEpisodeModal = false; // Đóng modal
        if (this.$toast) {
          this.$toast.success("Thêm tập phim thành công!"); // Hiển thị thông báo
        }
      } catch (error) {
        console.error("Thêm tập phim thất bại:", error);
        alert("Thêm tập phim thất bại!");
      }
    },

    closeAddEpisodeModal() {
      this.showAddEpisodeModal = false;
      this.newEpisode = {
        episodeNumber: '',
        title: '',
        linkFilmUrl: ''
      };
    },

    openEditEpisodeModal(episode) {
      this.editingEpisode = { ...episode }; // Sao chép dữ liệu tập phim để chỉnh sửa
      this.showEditEpisodeModal = true;
    },

    async updateEpisode() {
      try {
        const episodeData = {
          episodeId: this.editingEpisode.episodeId,
          seriesId: this.editingEpisode.seriesId,
          episodeNumber: this.editingEpisode.episodeNumber,
          title: this.editingEpisode.title,
          linkFilmUrl: this.editingEpisode.linkFilmUrl
        };

        // Kiểm tra xem seriesId có hợp lệ không
        if (episodeData.seriesId === null) {
          alert("Vui lòng chọn một series hợp lệ.");
          return; // Dừng lại nếu seriesId không hợp lệ
        }

        // Gửi yêu cầu PUT với dữ liệu JSON
        const response = await axios.put(`http://localhost:5289/api/AdminEpisode/UpdateEpisode/${episodeData.episodeId}`, episodeData, {
          headers: { "Content-Type": "application/json-patch+json" }
        });

        // Kiểm tra phản hồi từ server
        if (response.status === 204) {
          // Hiển thị thông báo sửa thành công
          if (this.$toast) {
            this.$toast.success("Tập phim đã được sửa thành công!");
          }

          // Cập nhật danh sách tập phim
          const series = this.seriesList.find(s => s.seriesId === episodeData.seriesId);
          if (series) {
            const updatedEpisodeIndex = series.episodes.findIndex(episode => episode.episodeId === episodeData.episodeId);
            if (updatedEpisodeIndex !== -1) {
              // Cập nhật thông tin tập phim trong danh sách
              series.episodes[updatedEpisodeIndex] = { ...series.episodes[updatedEpisodeIndex], ...episodeData };
            }
          }

          this.showEditEpisodeModal = false; // Đóng modal
        } else {
          alert("Cập nhật tập phim thất bại!");
        }
      } catch (error) {
        console.error("Cập nhật tập phim thất bại:", error);
        alert("Cập nhật tập phim thất bại!");
      }
    },
    closeEditEpisodeModal() {
      this.showEditEpisodeModal = false;
      this.editingEpisode = {};
    },

    deleteEpisode(episodeId, seriesId) {
      if (confirm("Bạn có chắc muốn xóa tập phim này không?")) {
        axios.delete(`http://localhost:5289/api/AdminEpisode/DeleteEpisode/${episodeId}`)
          .then(() => {
            const series = this.seriesList.find(s => s.seriesId === seriesId);
            if (series) {
              series.episodes = series.episodes.filter(e => e.episodeId !== episodeId);
            }
          })
          .catch(error => {
            console.error("Lỗi khi xóa tập phim:", error);
          });
      }
    },

    async fetchActorOptions() {
      // Gọi API để lấy danh sách diễn viên
      try {
        const response = await axios.get("http://localhost:5289/api/AdminActor", {
          params: {
            search: "",
            sortBy: "ActorId",
            sortDirection: "asc",
            page: 1,
            pageSize: 50, // Hiển thị tất cả diễn viên trong một lần tải
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
    async fetchcategorieOptions() {
      // Gọi API để lấy danh sách thể loại
      try {
        const response = await axios.get("http://localhost:5289/api/AdminCategories", {
          params: {
            search: "",
            sortBy: "CategoryId",
            sortDirection: "asc",
            page: 1,
            pageSize: 50, // Hiển thị tất cả diễn viên trong một lần tải
          },
        });

        this.categorieOptions = response.data.map((categorie) => ({
          id: categorie.categoryId,
          name: categorie.categoryName,
        }));
      } catch (error) {
        console.error("Lỗi khi tải danh sách diễn viên:", error);
      }
    },
    async fetchdirectorOptions() {
      // Gọi API để lấy danh sách đạo diễn
      try {
        const response = await axios.get("http://localhost:5289/api/AdminDirectors/List-Directors", {
          params: {
            search: "",
            sortBy: "DirectorId",
            sortDirection: "asc",
            page: 1,
            pageSize: 50, // Hiển thị tất cả diễn viên trong một lần tải
          },
        });

        this.directorOptions = response.data.map((director) => ({
          id: director.directorID,
          name: director.nameDir,
        }));
      } catch (error) {
        console.error("Lỗi khi tải danh sách diễn viên:", error);
      }
    },
    watch: {
      showSingleMovieForm(newVal) {
        if (newVal) {
          console.log(1);
          this.fetchdirectorOptions();
        }
      },
      showSeriesMovieForm(newVal) {
        if (newVal) {
          console.log("showSeriesMovieForm triggered");
          this.fetchActorOptions();
          this.fetchcategorieOptions();
          this.fetchdirectorOptions();
        }
      },
      showUpdateSeriesMovieForm(newVal) {
        if (newVal) {
          this.fetchActorOptions();
        }
      }
    },
    // series
    onAvatarChange(event) {
      const file = event.target.files[0];
      if (file) {
        this.avatarPreview = URL.createObjectURL(file);
        this.seriesMovieForm.avatarFile = event.target.files[0];
      } else {
        this.avatarPreview = null;
      }
    },
    onPosterChange(event) {
      console.log(event.target.files);
      const file = event.target.files[0];
      if (file) {
        this.posterPreview = URL.createObjectURL(file);
        this.seriesMovieForm.posterFile = event.target.files[0];
      } else {
        this.posterPreview = null;
      }
    },
    onAvatarChangee(event) {
      const file = event.target.files[0];
      if (file) {
        this.avatarPreview = URL.createObjectURL(file);
        this.seriesUpdateMovieForm.avatarFile = event.target.files[0];
      } else {
        this.avatarPreview = null;
      }
    },
    onPosterChangee(event) {
      console.log(event.target.files);
      const file = event.target.files[0];
      if (file) {
        this.posterPreview = URL.createObjectURL(file);
        this.seriesUpdateMovieForm.posterFile = event.target.files[0];
      } else {
        this.posterPreview = null;
      }
    },
    async fetchMovies() {
      this.loading = true;
      this.error = null;
      try {
        const response = await axios.get("http://localhost:5289/api/AdminMovies", {
          params: {
            sortBy: "Title",
            search: this.searchQuery.trim()
          },
        });
        this.movieList = (Array.isArray(response.data) ? response.data : []).sort(
          (a, b) => a.movieId - b.movieId
        );


        // Store all movies fetched from API
        this.allMovies = Array.isArray(response.data) ? response.data : [];

        // Calculate total pages based on items per page
        this.totalMoviePages = Math.ceil(this.movieList.length / this.itemsPerPage);

        // Update paginated data for the current page
        this.updateMoviePagination();
      } catch (error) {
        console.error("Lỗi khi tải danh sách phim:", error.response?.data?.message || error.message);
        this.error = "Không thể tải danh sách phim!";
      } finally {
        this.loading = false;
      }
    },
    updateMoviePagination() {
      const startIndex = (this.page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      this.paginatedMovies = this.movieList.slice(startIndex, endIndex);
    },

    async searchMovies() {
      this.page = 1; // Đặt lại về trang đầu tiên
      await this.fetchMovies(); // Gọi lại API với từ khóa tìm kiếm
    },
    async fetchSeries() {
      this.loading = true;
      this.error = null;
      try {
        const response = await axios.get("http://localhost:5289/api/AdminSeries", {
          params: {
            sortBy: "Title",
            search: this.searchQuery.trim(),
          },
        });

        // Khởi tạo sẵn showEpisodes và episodes cho từng series
        this.seriesList = (Array.isArray(response.data) ? response.data : [])
          .map(s => ({
            ...s,
            showEpisodes: false,
            episodes: []
          }))
          .sort((a, b) => a.seriesId - b.seriesId);

        // Store all series fetched from API
        this.allSeries = Array.isArray(response.data) ? response.data : [];

        // Tính toán tổng số trang
        this.totalSeriesPages = Math.ceil(this.allSeries.length / this.itemsPerPage);

        // Cập nhật dữ liệu phân trang cho trang hiện tại
        this.updateSeriesPagination();
      } catch (error) {
        console.error(
          "Lỗi khi tải danh sách phim bộ:",
          error.response?.data?.message || error.message
        );
        this.error = "Không thể tải danh sách phim bộ!";
      } finally {
        this.loading = false;
      }
    },

    updateSeriesPagination() {
      const startIndex = (this.page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      this.paginatedSeries = this.seriesList.slice(startIndex, endIndex);
    },
    // Thay đổi trang
    changePage(newPage) {
      if (newPage >= 1 && newPage <= this.totalMoviePages) {
        this.page = newPage; // Cập nhật trang hiện tại
      }
    },

    selectTab(tab) {
      this.currentTab = tab;
      this.searchQuery = "";
      this.page = 1;
      if (tab === "movies") {
        this.fetchMovies();
      } else if (tab === "series") {
        this.fetchSeries();
      }
    },

  },
  mounted() {
    this.fetchMovies(); // Lấy danh sách phim
    this.fetchSeries();
    this.fetchActorOptions();
    this.fetchcategorieOptions();
    this.fetchdirectorOptions();
  },
};
</script>
<style scoped>
@import "/src/assets/css/admin.css";

/* Modal Container */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  /* Nền tối mờ */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  transition: opacity 0.3s ease-in-out;
}

/* Modal Content */
.modal-content {
  background-color: #fff;
  padding: 40px;
  border-radius: 10px;
  width: 500px;
  /* Tăng chiều rộng */
  max-width: 100%;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease-in-out;
}

/* Tiêu đề Modal */
.modal-content h3 {
  font-size: 26px;
  margin-bottom: 30px;
  text-align: center;
  color: #333;
}

/* Label cho các trường nhập liệu */
.modal-content label {
  display: block;
  font-size: 16px;
  margin-bottom: 10px;
  color: #555;
  font-weight: 600;
}

/* Các trường nhập liệu */
.modal-content input {
  width: 100%;
  padding: 12px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 8px;
  margin-bottom: 25px;
  outline: none;
  transition: border-color 0.3s ease;
}

/* Hiệu ứng khi focus vào input */
.modal-content input:focus {
  border-color: #4e9ed6;
}

/* Phần chứa các nút Thêm và Hủy */
.modal-content .buttons-container {
  display: flex;
  justify-content: space-between;
  /* Đảm bảo nút cách nhau đều */
  gap: 10px;
  /* Khoảng cách giữa các nút */
}

/* Nút Thêm Tập Phim và Hủy */
.modal-content button {
  padding: 15px 25px;
  font-size: 18px;
  border-radius: 8px;
  cursor: pointer;
  border: none;
  transition: background-color 0.3s ease;
  width: 48%;
  /* Mỗi nút chiếm 48% chiều rộng của modal */
}

/* Nút Thêm Tập Phim */
.modal-content button:first-of-type {
  background-color: #4e9ed6;
  color: white;
}

.modal-content button:first-of-type:hover {
  background-color: #357ab7;
}

/* Nút Hủy */
.modal-content button:last-of-type {
  background-color: #ccc;
  color: #333;
}

.modal-content button:last-of-type:hover {
  background-color: #bbb;
}

/* Điều chỉnh cho màn hình nhỏ */
@media (max-width: 480px) {
  .modal-content {
    width: 90%;
  }

  .modal-content button {
    width: 100%;
    margin-top: 10px;
  }
}


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
