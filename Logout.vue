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

      <button @click="showForm = true" class="add-button">Thêm Phim</button>

      <!-- Form Thêm Phim -->
      <div v-if="showForm" class="form-overlay">
        <div class="form-container">
          <h2>{{ movieForm.editing ? 'Chỉnh sửa Phim' : 'Thêm Phim Mới' }}</h2>
          <form @submit.prevent="submitMovie">
            <div class="form-group">
              <label>ID</label>
              <input type="text" v-model="movieForm.movieId" :disabled="movieForm.editing" required />
            </div>
            <div class="form-group">
              <label>Tên phim</label>
              <input type="text" v-model="movieForm.title" required />
            </div>
            <div class="form-group">
              <label>Đạo diễn</label>
              <input type="text" v-model="movieForm.director" required />
            </div>
            <div class="form-group">
              <label>Thể loại</label>
              <input type="text" v-model="movieForm.genre" required />
            </div>
            <div class="form-group">
              <label>Rating</label>
              <input type="text" v-model="movieForm.rating" required />
            </div>
            <div class="form-group">
              <label>Trạng thái</label>
              <select v-model="movieForm.status" required>
                <option value="Công chiếu">Công chiếu</option>
                <option value="Sắp ra mắt">Sắp ra mắt</option>
              </select>
            </div>
            <button type="submit" class="submit-button">{{ movieForm.editing ? 'Cập nhật' : 'Thêm Phim' }}</button>
            <button type="button" class="cancel-button" @click="cancelForm">Hủy</button>
          </form>
        </div>
      </div>

      <!-- Danh sách phim -->
      <div>
        <h2>Danh Sách Phim</h2>
        <table class="data-table">
          <thead>
            <tr>
              <th>ID</th>
              <th>Tên phim</th>
              <th>Đạo diễn</th>
              <th>Thể loại</th>
              <th>Rating</th>
              <th>Trạng thái</th>
              <th>Hành động</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(movie, index) in movieList" :key="movie.movieId">
              <td>{{ movie.movieId }}</td>
              <td>{{ movie.title }}</td>
              <td>{{ movie.director }}</td>
              <td>{{ movie.genre }}</td>
              <td>{{ movie.rating }}</td>
              <td>{{ movie.status }}</td>
              <td>
                <button @click="editMovie(index)" class="edit-button">Sửa</button>
                <button @click="deleteMovie(index)" class="delete-button">Xóa</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showForm: false,
      movieForm: {
        movieId: '',
        title: '',
        director: '',
        genre: '',
        rating: '',
        status: 'Công chiếu',
        editing: false,
      },
      movieList: [],
    };
  },
  methods: {
    submitMovie() {
      if (this.movieForm.editing) {
        const index = this.movieList.findIndex(movie => movie.movieId === this.movieForm.movieId);
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
    deleteMovie(index) {
      this.movieList.splice(index, 1);
    },
    cancelForm() {
      this.movieForm = {
        movieId: '',
        title: '',
        director: '',
        genre: '',
        rating: '',
        status: 'Công chiếu',
        editing: false,
      };
      this.showForm = false;
    },
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
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

/* Form Container */
.form-container {
  background: #ffffff;
  padding: 20px;
  border-radius: 10px;
  width: 400px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
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

.tabs button, .add-button {
  padding: 12px 50px;
  font-size: 18px;
  background-color: #f0f0f0;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.tabs button.active, .add-button{
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
</style>