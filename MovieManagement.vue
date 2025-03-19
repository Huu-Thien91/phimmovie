<template>
<div class="container">
  <aside class="sidebar">
    <h2>Bảng điều khiển</h2>
    <ul>
      <li><router-link to="/admin/movies">Quản lý phim</router-link></li>
      <li><router-link to="/admin/finance">Quản lí tài chính</router-link></li>
      <li><router-link to="/admin/contentmanagement">Quản lí nội dung</router-link></li>
      <li><router-link to="/admin/vipmanagenment">Quản lí tài khoản VIP và Thanh Toán</router-link></li>
      <li><router-link to="/admin/account">Quản lí hệ thống và bảo mật (admin)</router-link></li>
      <li><router-link to="/admin/user">Quản lí người dùng</router-link></li>
      <li><router-link to="/admin/statisticsandreports">Thống kê và báo cáo</router-link></li>
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

      <!-- Search Bar -->
      <div class="search-bar">
        <input type="text" v-model="searchQuery" placeholder="Tìm kiếm phim..." />
      </div>

      <!-- Quản lý phim lẻ -->
      <div v-if="currentTab === 'movies'" class="tab-content">
        <h2>Quản lý Phim Lẻ</h2>
        <table class="data-table">
          <thead>
            <tr>
              <th>ID</th>
              <th>Ảnh Phim</th>
              <th>Tiêu đề</th>
              <th>Đạo diễn</th>
              <th>Thể loại</th>
              <th>Rating</th>
              <th>Trạng thái</th>
              <th>Hành động</th>
              
            </tr>
          </thead>
          <tbody>
            <tr v-for="(movie, index) in filteredMovies" :key="movie.id">
              <td>{{ movie.id }}</td>
              <td>{{ movie.image }}</td>
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
        <form @submit.prevent="submitMovie" class="movie-form">
          <div class="form-group" v-for="(value, key) in movieForm" :key="key">
            <label>{{ formLabels[key] }}</label>
            <input v-if="key !== 'status' && key !== 'editing' && key !== 'id'" type="text" v-model="movieForm[key]"
              required />
            <select v-else-if="key === 'status'" v-model="movieForm.status" required>
              <option value="Công chiếu">Công chiếu</option>
              <option value="Sắp ra mắt">Sắp ra mắt</option>
            </select>
          </div>
          <button type="submit" class="submit-button">{{ movieForm.editing ? 'Cập nhật' : 'Thêm Phim Lẻ' }}</button>
        </form>
      </div>

      <!-- Quản lý phim bộ -->
      <div v-if="currentTab === 'series'" class="tab-content">
        <h2>Quản lý Phim Bộ</h2>
        <table class="data-table">
          <thead>
            <tr>
              <th>ID</th>
              <th>Ảnh Phim</th>
              <th>Tiêu đề</th>
              <th>Đạo diễn</th>
              <th>Thể loại</th>
              <th>Rating</th>
              <th>Trạng thái</th>
              <th>Hành động</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(series, index) in filteredSeries" :key="series.id">
              <td>{{ series.id }}</td>
              <td>{{ series.image }}</td>
              <td>{{ series.title }}</td>
              <td>{{ series.director }}</td>
              <td>{{ series.genre }}</td>
              <td>{{ series.rating }}</td>
              <td>{{ series.status }}</td>
              <td>
                <button @click="editSeries(index)" class="edit-button">Sửa</button>
                <button @click="deleteSeries(index)" class="delete-button">Xóa</button>
              </td>
            </tr>
          </tbody>
        </table>

        <form @submit.prevent="submitSeries" class="movie-form">
          <div class="form-group" v-for="(value, key) in seriesForm" :key="key">
            <label>{{ formLabels[key] }}</label>
            <input v-if="key !== 'status' && key !== 'editing' && key !== 'id'" type="text" v-model="seriesForm[key]"
              required />
            <select v-else-if="key === 'status'" v-model="seriesForm.status" required>
              <option value="Đang phát sóng">Đang phát sóng</option>
              <option value="Hoàn tất">Hoàn tất</option>
            </select>
          </div>
          <button type="submit" class="submit-button">{{ seriesForm.editing ? 'Cập nhật' : 'Thêm Phim Bộ' }}</button>
        </form>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      currentTab: 'movies',
      searchQuery: '',
      formLabels: {
        title: 'Tiêu đề',
        director: 'Đạo diễn',
        genre: 'Thể loại',
        rating: 'Rating',
        status: 'Trạng thái',
      },
      movieForm: {
        title: '',
        director: '',
        genre: '',
        rating: '',
        status: 'Công chiếu',
        editing: false,
        id: null,
      },
      seriesForm: {
        title: '',
        director: '',
        genre: '',
        rating: '',
        status: 'Đang phát sóng',
        editing: false,
        id: null,
      },
      movieList: [
        { id: 1, title: 'Avengers', director: 'Joss Whedon', genre: 'Action', rating: 8.5, status: 'Công chiếu' },
        { id: 2, title: 'Inception', director: 'Christopher Nolan', genre: 'Sci-Fi', rating: 9.0, status: 'Công chiếu' }
      ],
      seriesList: [
        { id: 1, title: 'Breaking Bad', director: 'Vince Gilligan', genre: 'Crime', rating: 9.5, status: 'Hoàn tất' },
        { id: 2, title: 'Game of Thrones', director: 'David Benioff & D.B. Weiss', genre: 'Fantasy', rating: 8.0, status: 'Hoàn tất' }
      ],
    };
  },
  computed: {
    filteredMovies() {
      return this.movieList.filter(movie => movie.title.toLowerCase().includes(this.searchQuery.toLowerCase()));
    },
    filteredSeries() {
      return this.seriesList.filter(series => series.title.toLowerCase().includes(this.searchQuery.toLowerCase()));
    }
  },
  methods: {
    selectTab(tab) {
      this.currentTab = tab;
    },
    goBack() {
      this.$router.go(-1);
    },
    submitMovie() {
      if (this.movieForm.editing) {
        const index = this.movieList.findIndex(movie => movie.id === this.movieForm.id);
        this.movieList.splice(index, 1, { ...this.movieForm });
      } else {
        this.movieForm.id = this.movieList.length + 1;
        this.movieList.push({ ...this.movieForm });
      }
      this.resetMovieForm();
    },
    submitSeries() {
      if (this.seriesForm.editing) {
        const index = this.seriesList.findIndex(series => series.id === this.seriesForm.id);
        this.seriesList.splice(index, 1, { ...this.seriesForm });
      } else {
        this.seriesForm.id = this.seriesList.length + 1;
        this.seriesList.push({ ...this.seriesForm });
      }
      this.resetSeriesForm();
    },
    editMovie(index) {
      this.movieForm = { ...this.movieList[index], editing: true };
    },
    editSeries(index) {
      this.seriesForm = { ...this.seriesList[index], editing: true };
    },
    deleteMovie(index) {
      this.movieList.splice(index, 1);
    },
    deleteSeries(index) {
      this.seriesList.splice(index, 1);
    },
    resetMovieForm() {
      this.movieForm = {
        title: '',
        director: '',
        genre: '',
        rating: '',
        status: 'Công chiếu',
        editing: false,
        id: null,
      };
    },
    resetSeriesForm() {
      this.seriesForm = {
        title: '',
        director: '',
        genre: '',
        rating: '',
        status: 'Đang phát sóng',
        editing: false,
        id: null,
      };
    }
  }
};
</script>

<style scoped>
@import "/src/assets/css/admin.css";

.movie-management {
  background-color: #fff;
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  animation: fadeIn 1s ease-in-out;
}

.tabs {
  display: flex;
  gap: 20px;
}

.tabs button {
  padding: 12px 50px;
  font-size: 18px;
  background-color: #f0f0f0;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.tabs button.active {
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