<template>
  <div class="container">
    <aside class="sidebar">
      <h2>Bảng điều khiển</h2>
      <ul>
        <li><router-link to="/admin">Dashboard</router-link></li>
        <li><router-link to="/admin/movies">Quản lý phim</router-link></li>
        <li><router-link to="/admin/vipmanagenment">Quản lí tài khoản VIP và Thanh Toán</router-link></li>
        <li><router-link to="/admin/account">Quản lí hệ thống và bảo mật (admin)</router-link></li>
        <li><router-link to="/admin/user">Quản lí người dùng</router-link></li>
        <li><router-link to="/admin/transactions">Lịch sử giao dịch</router-link></li>
        <li><router-link to="/admin/setting">Cài đặt chung</router-link></li>
        <li><router-link to="/login">Đăng xuất</router-link></li>
      </ul>
    </aside>
    <div class="movie-management">
      <h1>🎥 Quản lý Phim</h1>

      <!-- Tabs -->
      <div class="tabs">
        <button @click="selectTab('movies')" :class="{ active: currentTab === 'movies' }">Phim Lẻ</button>
        <button @click="selectTab('series')" :class="{ active: currentTab === 'series' }">Phim Bộ</button>
      </div>

      <!-- Danh sách phim -->
      <div v-if="currentTab === 'movies'" class="tab-content">
        <h2>Quản lý Phim Lẻ</h2>
        <button @click="showForm = true" class="add-button">Thêm Phim</button>
        <div class="search-bar">
          <input v-model="searchQuery" type="text" placeholder="Tìm kiếm phim lẻ" @input="searchMovies" />
        </div>
        <div>
          <ul>
            <li v-for="movie in filteredMovies" :key="movie.id">
              {{ movie.title }}
            </li>
          </ul>
        </div>
      </div>
      <table class="data-table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Avatar</th>
            <th>Poster</th>
            <th>Tên phim</th>
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
          <tr v-if="movieList.length === 0 && !loading">
            <td colspan="14">Không có phim nào phù hợp!</td>
          </tr>
          <tr v-for="(movie, index) in paginatedMovies" :key="movie.movieId">
            <td>{{ movie.movieId }}</td>
            <td><img :src="movie.avatarUrl" alt="Avatar" width="100" /></td>
            <td><img :src="movie.posterUrl" alt="Poster" width="100" /></td>
            <td>{{ movie.title }}</td>
            <td>
              <div v-for="actor in movie.actors" :key="actor.actorId">
                {{ actor.nameAct }}
              </div>
            </td>
            <td>{{ movie.director }}</td>
            <td>
              <div v-for="category in movie.categories" :key="category.categoryId">
                {{ category.categoryName }}
              </div>
            </td>
            <td>{{ movie.nation || "N/A" }}</td>
            <td>{{ movie.yearReleased || "N/A" }}</td>
            <td>{{ movie.description || "N/A" }}</td>
            <td>{{ movie.isHot ? "Có" : "Không" }}</td>
            <td>{{ movie.rating || "N/A" }}</td>
            <td>{{ movie.status || "N/A" }}</td>
            <td>
              <button @click="editMovie(index)" class="edit-button">Sửa</button>
              <button @click="deleteMovie(index)" class="delete-button">Xóa</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <div v-if="currentTab === 'series'" class="tab-content">
      <h2>Quản lý Phim Bộ</h2>
      <button @click="showForm = true" class="add-button">Thêm Phim Bộ</button>

      <!-- Bảng hiển thị Phim Bộ -->
      <table class="data-table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Tên phim</th>
            <th>Avatar</th>
            <th>Poster</th>
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
          <tr v-if="seriesList.length === 0 && !loading">
            <td colspan="14">Không có phim bộ nào phù hợp!</td>
          </tr>
          <tr v-for="(series, index) in seriesList" :key="series.seriesId">
            <td>{{ series.seriesId }}</td>
            <td>{{ series.title }}</td>
            <td><img :src="series.avatarUrl" alt="Avatar" width="100" /></td>
            <td><img :src="series.posterUrl" alt="Poster" width="100" /></td>
            <td>{{ series.directorId || "Chưa cập nhật" }}</td>
            <td>{{ series.categoriesIds ? series.categoriesIds.join(", ") : "Chưa cập nhật" }}</td>
            <td>{{ series.nation || "Chưa cập nhật" }}</td>
            <td>{{ series.season || "N/A" }}</td>
            <td>{{ series.yearReleased || "N/A" }}</td>
            <td>{{ series.description || "N/A" }}</td>
            <td>{{ series.isHot ? "Có" : "Không" }}</td>
            <td>{{ series.rating || "N/A" }}</td>
            <td>{{ series.status === 1 ? "Đang hoạt động" : "Không hoạt động" }}</td>
            <td>
              <button @click="editSeries(index)" class="edit-button">Sửa</button>
              <button @click="deleteSeries(index)" class="delete-button">Xóa</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <!-- Nút điều hướng phân trang -->
    <div class="pagination">
      <button @click="changePage(page - 1)" :disabled="page === 1">Trước</button>
      <span>Trang {{ page }} / {{ totalPages }}</span>
      <button @click="changePage(page + 1)" :disabled="page === totalPages">Sau</button>
    </div>


    <!-- Form Thêm/Sửa Phim -->
    <div v-if="showForm" class="form-overlay">
      <div class="form-container">
        <h2>{{ movieForm.editing ? 'Chỉnh sửa Phim' : 'Thêm Phim Mới' }}</h2>
        <form @submit.prevent="submitMovie" class="movie-form">
          <div class="form-columns">
            <div class="form-column left-column">
              <div class="form-group">
                <label>Tên phim</label>
                <input type="text" v-model="movieForm.title" required />
              </div>
              <div class="form-group">
                <label>Diễn viên</label>
                <multiselect id="actor-search" v-model="movieForm.actors" tag-placeholder="Thêm diễn viên"
                  placeholder="Tìm kiếm hoặc thêm diễn viên" label="name" track-by="code" :options="actorOptions"
                  :multiple="true" :taggable="true">
                </multiselect>
              </div>


              <div class="form-group">
                <label>Đạo diễn</label>
                <input type="text" v-model="movieForm.director" required />
              </div>
              <div class="form-group">
                <label>Năm Phát Hành</label>
                <input type="text" v-model="movieForm.yearReleased" required />
              </div>
              <div class="form-group">
                <label>Trạng thái</label>
                <select v-model="movieForm.status" required>
                  <option value="Công chiếu">Công chiếu</option>
                  <option value="Sắp ra mắt">Sắp ra mắt</option>
                </select>
              </div>
              <div class="form-group">
                <label>Mô tả</label>
                <input type="text" v-model="movieForm.description" required />
              </div>
              <div class="form-group">
                <label>Hot</label>
                <select v-model="movieForm.isHot" required>
                  <option value="true">True</option>
                  <option value="false">False</option>
                </select>
              </div>
            </div>

            <div class="form-column right-column">
              <div class="form-group">
                <label>Rating</label>
                <input type="number" v-model="movieForm.rating" required min="0" max="10" />
              </div>
              <div class="form-group">
                <label>Thể Loại</label>
                <multiselect id="categorie-search" v-model="optionName" tag-placeholder="Thêm thể loại"
                  placeholder="Tìm kiếm hoặc thêm thể loại" label="name" track-by="code" :options="categorieOptions"
                  :multiple="true" :taggable="true">
                </multiselect>
              </div>
              <div class="form-group">
                <label>Quốc gia</label>
                <input type="text" v-model="movieForm.nation" required />
              </div>
              <div class="form-group">
                <label>Avatar</label>
                <input type="file" @change="onAvatarChange" accept="image/*" />
                <img v-if="avatarPreview" :src="avatarPreview" alt="Avatar Preview"
                  style="width: 150px; height: auto; margin-top: 10px;" />
              </div>

              <div class="form-group">
                <label>Poster</label>
                <input type="file" @change="onPosterChange" accept="image/*" />
              </div>
              <div class="form-group">
                <label>Link Film</label>
                <input type="url" v-model="movieForm.linkFilmUrl" required placeholder="URL của phim" />
              </div>
            </div>
          </div>

          <div class="button-group">
            <button type="submit" class="submit-button">
              {{ movieForm.editing ? 'Cập nhật' : 'Thêm Phim' }}
            </button>
            <button type="button" class="cancel-button" @click="cancelForm">Hủy</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Multiselect from "vue-multiselect";
import "vue-multiselect/dist/vue-multiselect.min.css";
import { ref } from "vue";
const currentTab = ref("movies"); // Tab mặc định là "Phim Lẻ"
const paginatedMovies = ref([]); // Dữ liệu phim lẻ
const paginatedSerie = ref([]); // Dữ liệu phim bộ
const selectTab = (tab) => {
  currentTab.value = tab; // Chuyển đổi tab
};

export default {
  components: {
    Multiselect,
  },
  data() {
    return {
      searchQuery: '',
      movies: [],
      loading: false,
      error: null,
      avatarPreview: null,
      currentTab: "movies",
      showForm: false,
      movieList: [],
      seriesList: [],
      totalPages: 0,
      totalSeriesPages: 0,
      page: 1, // Trang hiện tại
      itemsPerPage: 5, // 5 phim trên mỗi trang
      optionName: [{ name: "Hành Động", code: "js" }],
      categorieOptions: [
        { name: "Hài Kịch", code: "vu" },
        { name: "Tình Cảm", code: "js" },
        { name: "Kinh Dị", code: "os" },
      ],
      movieForm: {
        movieId: "",
        title: "",
        director: "",
        yearReleased: "",
        status: "",
        actors: [],
        categories: [],
        rating: "",
        status: "Công chiếu",
        editing: false,
        isHot: "false", // Default value for Hot
        linkFilm: "", // New field for film link
        posterFile: null, // File poster
        avatarFile: null, // File đại diện
        description: "", // Mô tả phim
      },
      actorOptions: []
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.movieList.length / this.itemsPerPage); // Tổng số trang
    },
    paginatedMovies() {
      const start = (this.page - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.movieList.slice(start, end);
    },
    paginatedSeries() {
      const start = (this.page - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.seriesList.slice(start, end);
    },
  },
  methods: {
    async fetchActorOptions() {
      // Gọi API để lấy danh sách diễn viên
      try {
        const response = await axios.get("http://localhost:5289/api/AdminActor");
        this.actorOptions = data.map(actor => ({
          name: actor.nameAct, // Use 'nameAct' for actor name
          code: actor.actorId  // Use 'actorId' as the unique identifier
        }));
        console.log("this.actors: ", this.actors);
      } catch (error) {
        console.error("Lỗi khi tải danh sách diễn viên:", error);
      }
    },
    onAvatarChange(event) {
      const file = event.target.files[0];
      if (file) {
        this.avatarPreview = URL.createObjectURL(file);
        console.log("URL của ảnh:", this.avatarPreview);
      } else {
        this.avatarPreview = null;
      }
    },
    async addFilm() {
      try {
        // Chuẩn bị FormData để gửi dữ liệu `multipart/form-data`
        const formData = new FormData();
        formData.append("actorIds", this.movieForm.actors.map((actor) => actor.id).join(","));
        formData.append("IsHot", this.movieForm.isHot ? "true" : "false");
        formData.append("DirectorId", this.movieForm.director);
        formData.append("posterFile", this.movieForm.posterFile);
        formData.append("LinkFilmUrl", this.movieForm.linkFilm);
        formData.append("AvatarUrlFile", this.movieForm.avatarFile);
        formData.append("Title", this.movieForm.title);
        formData.append("YearReleased", this.movieForm.yearReleased);
        formData.append("categoryIds", this.movieForm.categories.map((cat) => cat.id).join(","));
        formData.append("Description", this.movieForm.description);
        formData.append("Rating", this.movieForm.rating);

        // Gửi yêu cầu đến API
        const response = await axios.post("http://localhost:26762/api/Movie/AddFilm", formData, {
          headers: {
            "Content-Type": "multipart/form-data",
          },
        });

        console.log("Phim được thêm thành công:", response.data);
        alert("Thêm phim thành công!");
        this.fetchMovies(); // Cập nhật danh sách phim
      } catch (error) {
        console.error("Lỗi khi thêm phim:", error.response?.data || error.message);
        alert("Đã xảy ra lỗi khi thêm phim.");
      }
    }
    ,
    async fetchMovies(page = 1) {
      this.loading = true;
      this.error = null;
      try {
        const response = await axios.get("http://localhost:5289/api/AdminMovie", {
          params: { page }, // Add pagination parameter if needed
        });
        this.movieList = response.data; // Assuming response.data contains the list of movies
        this.totalPages = response.totalPages; // Adjust based on your API response structure
      } catch (error) {
        console.error("Lỗi khi tải phim:", error);
        this.error = 'Không thể tải danh sách phim!';
      } finally {
        this.loading = false;
      }
    },

    // Thay đổi trang
    changePage(newPage) {
      if (
        newPage >= 1 &&
        newPage <=
        (this.currentTab === "movies" ? this.totalPages : this.totalSeriesPages)
      ) {
        this.page = newPage;
        this.fetchData(this.currentTab === "movies" ? "movies" : "series", newPage);
      }
    },

    async getMovieDetails(movieId) {
      this.loading = true;
      this.error = null;

      try {
        const response = await axios.get(`http://localhost:5289/api/Movie/Search/${movieId}`);

        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }

        const movieDetails = await response.json();
        this.loading = false;
        return movieDetails;
      } catch (err) {
        this.error = err.message;
        this.loading = false;
      }
    },
    async deleteMovie(index) {
      if (confirm("Bạn có chắc muốn xóa phim này?")) {
        try {
          const movieId = this.movieList[index].movieId;
          await axios.delete(`http://localhost:26762/api/Movie/de/${movieId}`);
          this.fetchMovies();
        } catch (error) {
          console.error("Lỗi khi xóa phim:", error);
        }
      }
    },
    async fetchSeries(page = 1) {
      this.loading = true;
      this.error = null;
      try {
        const response = await axios.get("http://localhost:5289/api/Serie", {
          params: {
            sortBy: "Title",
            sortDirection: "asc",
            page,
            pageSize: 5, // Số phim bộ mỗi trang
          },
        });
        this.seriesList = response.data.series || []; // Danh sách phim bộ
        this.totalSeriesPages = response.data.totalPages || 0; // Tổng số trang
      } catch (error) {
        console.error("Lỗi khi tải phim bộ:", error.message);
        this.error = error.response?.data?.message || 'Không thể tải danh sách phim bộ!';
      } finally {
        this.loading = false;
      }
    },
    async searchMovies() {
      if (!this.searchQuery.trim()) {
        this.fetchMovies(1); // Tải danh sách phim lẻ nếu không có từ khóa
        return;
      }

      this.loading = true;
      this.error = null;

      try {
        const response = await axios.get("http://localhost:5289/api/AdminMovie/Seach", {
          params: { query: this.searchQuery, page: this.page, pageSize: this.itemsPerPage },
        });
        this.movieList = response.data.movies || [];
        this.totalPages = response.data.totalPages || 0;
      } catch (error) {
        console.error("Lỗi khi tìm kiếm phim lẻ:", error.message);
        this.error = "Không thể tìm kiếm dữ liệu phim lẻ!";
      } finally {
        this.loading = false;
      }
    },
    changePage(newPage) {
      if (newPage >= 1 && newPage <= this.totalPages) {
        this.page = newPage;
        if (this.searchQuery.trim()) {
          this.searchMovies(); // Tìm kiếm trên trang mới
        } else {
          this.fetchMovies(newPage); // Tải danh sách bình thường
        }
      }
    },
    submitMovie() {
      if (this.movieForm.editing) {
        const index = this.movieList.findIndex((movie) => movie.movieId === this.movieForm.movieId);
        this.movieList.splice(index, 1, { ...this.movieForm });
      } else {
        this.movieList.push({ ...this.movieForm });
      }
      this.cancelForm();
    },
    selectTab(tab) {
      this.currentTab = tab;
      this.page = 1; // Reset trang về 1 khi chuyển tab
      if (tab === "movies") {
        this.fetchMovies();
      } else if (tab === "series") {
        this.fetchSeries();
      }
    },

    editMovie(index) {
      this.movieForm = { ...this.movieList[index], editing: true };
      this.showForm = true;
    },
    addActor() {
      this.movieForm.actors.push({ nameAct: "" });
    },
    cancelForm() {
      this.movieForm = {
        movieId: "",
        title: "",
        director: "",
        yearReleased: "",
        status: "Công chiếu",
        actors: [],
        categories: [],
        rating: null,
        isHot: false,
        linkFilm: "",
        posterFile: null,
        avatarFile: null,
        description: "",
        editing: false,
      };
      this.showForm = false;
    },
  },
  mounted() {
    this.fetchMovies(); // Lấy danh sách phim
    this.fetchActorOptions();
    //this.fetchActorOptions(); // Tải danh sách diễn viên
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
.add-button {
  padding: 12px 50px;
  font-size: 18px;
  background-color: #f0f0f0;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.tabs button.active,
.add-button {
  background-color: #4caf50;
  color: white;
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
}

.add-button {
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