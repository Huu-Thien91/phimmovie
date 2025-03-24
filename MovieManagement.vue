<template>
  <div class="container">
    <aside class="sidebar">
      <h2>Bảng điều khiển</h2>
      <ul>
        <li><router-link to="/admin/movies">Quản lý phim</router-link></li>
        <li><router-link to="/admin/finance">Quản lí tài chính</router-link></li>
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
        <table class="data-table">
          <thead>
            <tr>
              <th>ID</th>
              <th>Tên phim</th>
              <th>Diễn viên</th>
              <th>Đạo diễn</th>
              <th>Thể loại</th>
              <th>Rating</th>
              <th>Trạng thái</th>
              <th>Hành động</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(movie, index) in paginatedMovies" :key="movie.movieId">
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
              <td>{{ movie.rating }}</td>
              <td>{{ movie.status || "N/A" }}</td>
              <td>
                <button @click="editMovie(index)" class="edit-button">Sửa</button>
                <button @click="deleteMovie(index)" class="delete-button">Xóa</button>
              </td>
            </tr>
          </tbody>
        </table>
        <!-- Nút điều hướng phân trang -->
        <div class="pagination">
          <button @click="changePage(page - 1)" :disabled="page === 1">Trước</button>
          <span>Trang {{ page }} / {{ totalPages }}</span>
          <button @click="changePage(page + 1)" :disabled="page === totalPages">Sau</button>
        </div>
      </div>

      <!-- Form Thêm/Sửa Phim -->
      <div v-if="showForm" class="form-overlay">
        <div class="form-container">
          <h2>{{ movieForm.editing ? 'Chỉnh sửa Phim' : 'Thêm Phim Mới' }}</h2>
          <form @submit.prevent="submitMovie">
            <div class="form-group">
              <label>Tên phim</label>
              <input type="text" v-model="movieForm.title" required />
            </div>
            <div class="form-group">
              <label>Diễn viên</label>
              <multiselect id="actor-search" v-model="optionValue" tag-placeholder="Thêm diễn viên"
                placeholder="Tìm kiếm hoặc thêm diễn viên" label="name" track-by="code" :options="actorOptions"
                :multiple="true" :taggable="true">
                <pre class="language-json"><code>{{ value }}</code></pre>
              </multiselect>
            </div>
            <div class="form-group">
              <label>Đạo diễn</label>
              <input type="text" v-model="movieForm.director" required />
            </div>
            <div class="form-group">
              <label>Thể Loại</label>
              <div v-for="category in availableCategories" :key="category.categoryId">
                <label>
                  <input type="checkbox" :value="category" v-model="movieForm.categories" />
                  {{ category.categoryName }}
                </label>
              </div>
            </div>
            <div class="form-group">
              <label>Rating</label>
              <input type="number" v-model="movieForm.rating" required min="0" max="10" />
            </div>
            <div class="form-group">
              <label>Trạng thái</label>
              <select v-model="movieForm.status" required>
                <option value="Công chiếu">Công chiếu</option>
                <option value="Sắp ra mắt">Sắp ra mắt</option>
              </select>
            </div>
            <button type="submit" class="submit-button">
              {{ movieForm.editing ? 'Cập nhật' : 'Thêm Phim' }}
            </button>
            <button type="button" class="cancel-button" @click="cancelForm">Hủy</button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Multiselect from "vue-multiselect";
import "vue-multiselect/dist/vue-multiselect.min.css";
export default {
  components: {
    Multiselect
  },
  methods: {
    // addTag(newTag) {
    //   const tag = {
    //     name: newTag,
    //     code: newTag.substring(0, 2) + Math.floor((Math.random() * 10000000))
    //   }
    //   this.options.push(tag)
    //   this.director.push(tag)
    // }
  },
  data() {
    return {
      currentTab: "movies",
      showForm: false,
      movieList: [],
      page: 1, // Trang hiện tại
      itemsPerPage: 5, // 5 phim trên mỗi trang
      availableCategories: [
        { categoryId: 1, categoryName: "Hành Động" },
        { categoryId: 2, categoryName: "Drama" },
        { categoryId: 3, categoryName: "Hài Kịch" },
        { categoryId: 6, categoryName: "Lãng Mạn" },
      ],
      movieForm: {
        movieId: "",
        title: "",
        director: "",
        actors: [],
        categories: [],
        rating: "",
        status: "Công chiếu",
        editing: false,
      },
      optionValue: [
        { name: 'Javascript', code: 'js' }
      ],
      actorOptions: [
        { name: 'Vue.js', code: 'vu' },
        { name: 'Javascript', code: 'js' },
        { name: 'Open Source', code: 'os' }
      ]
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.movieList.length / this.itemsPerPage); // Tổng số trang
    },
    paginatedMovies() {
      const start = (this.page - 1) * this.itemsPerPage; // Vị trí bắt đầu
      const end = start + this.itemsPerPage; // Vị trí kết thúc
      return this.movieList.slice(start, end);
    },
  },
  methods: {
    async fetchActorOptions() {
      // Gọi API để lấy danh sách diễn viên
      try {
        const response = await axios.get("http://localhost:26762/api/Actors");
        this.actors = response.data.map((actor) => ({
          actorId: actor.actorId,
          nameAct: actor.nameAct,
        }));
        console.log(" this.actors: ", this.actors)
      } catch (error) {
        console.error("Lỗi khi tải danh sách diễn viên:", error);
      }
    },
    addNewActor(newActorName) {
      // Thêm diễn viên mới vào danh sách (nếu không tìm thấy)
      const newActor = {
        actorId: `new-${Math.random()}`, // Tạo ID giả lập
        nameAct: newActorName,
      };
      this.actorOptions.push(newActor); // Thêm vào danh sách tùy chọn
      this.movieForm.actors.push(newActor); // Thêm vào diễn viên được chọn
    },
    async fetchMovies() {
      try {
        const response = await axios.get("http://localhost:26762/api/Movie");
        this.movieList = response.data;
      } catch (error) {
        console.error("Lỗi khi tải phim:", error);
      }
    },
    changePage(newPage) {
      if (newPage >= 1 && newPage <= this.totalPages) {
        this.page = newPage; // Thay đổi trang
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
    submitMovie() {
      if (this.movieForm.editing) {
        const index = this.movieList.findIndex(
          (movie) => movie.movieId === this.movieForm.movieId
        );
        this.movieList.splice(index, 1, { ...this.movieForm });
      } else {
        this.movieList.push({ ...this.movieForm });
      }
      this.cancelForm();
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
        actors: [],
        categories: [],
        rating: "",
        status: "Công chiếu",
        editing: false,
      };
      this.showForm = false;
    },
  },
  mounted() {
    this.fetchMovies(); // Lấy danh sách phim
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
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

/* Form Container */
.form-container {
  background: #fff;
  padding: 30px;
  /* Tăng khoảng cách bên trong */
  border-radius: 10px;
  /* Bo góc rõ hơn */
  width: 600px;
  /* Tăng chiều rộng */
  max-width: 90%;
  /* Giới hạn tối đa trong viewport */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  /* Làm form nổi bật hơn */
}

h2 {
  margin-bottom: 20px;
  font-size: 1.5rem;
  text-align: center;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input,
.form-group select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

h2 {
  margin-bottom: 20px;
  font-size: 1.5rem;
  text-align: center;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input,
.form-group select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.submit-button {
  width: 100%;
  background: #4caf50;
  color: #fff;
  border: none;
  padding: 10px;
  border-radius: 5px;
  cursor: pointer;
}

.cancel-button {
  width: 100%;
  background: #f44336;
  color: #fff;
  border: none;
  padding: 10px;
  margin-top: 10px;
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
</style>