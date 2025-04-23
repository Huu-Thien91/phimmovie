<template>
  <div class="container">
    <aside class="sidebar">
      <h2> Bảng điều khiển </h2>
      <ul>
        <li><router-link to="/">Quản lý phim</router-link></li>
        <li><router-link to="/admin/actors">Quản lí diễn viên </router-link></li>
        <li><router-link to="/admin/directors">Quản lí đạo diễn </router-link></li>
        <li><router-link to="/admin/categories">Thể Loại</router-link></li>
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

      <!-- Danh sách phim lẻ -->
      <div v-if="currentTab === 'movies'" class="tab-content">
        <h2>Quản lý Phim Lẻ</h2>
        <button @click="showSingleMovieForm = true" class="add-button">Thêm Phim Lẻ</button>
        <div class="search-bar">
          <input v-model="searchQuery" type="text" placeholder="Tìm kiếm phim lẻ" @input="searchMovies" />
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
              <th>Hành động</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="paginatedMovies.length === 0 && !loading">
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
              <td>{{ movie.director }}
                <div v-for="director in movie.directors" :key="director.directorID">
                  {{ director.nameDir }}
                </div>
              </td>
              <td>
                <div v-for="category in movie.categories" :key="category.categoryId">
                  {{ category.categoryName }}
                </div>
              </td>
              <td>{{ movie.nation }}</td>
              <td>{{ movie.yearReleased }}</td>
              <td>{{ movie.description }}</td>
              <td>{{ movie.isHot ? "Có" : "Không" }}</td>
              <td>{{ movie.rating }}</td>
              <td>
                <button @click="editMovie(index)" class="edit-button">Sửa</button>
                <button @click="deleteMovie(index)" class="delete-button">Xóa</button>
              </td>
            </tr>
          </tbody>
        </table>
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
                <td>{{ series.director }}
                  <div v-for="director in series.directors" :key="director.directorID">
                    {{ series.nameDir }}
                  </div>
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
                <td>
                  <button @click="toggleEpisodes(series)" class="view-button">
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
                          <th>Link</th>
                          <th>Hành động</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="episode in series.episodes" :key="episode.episodeId">
                          <td>Tập {{ episode.episodeNumber }}</td>
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
      <!-- Nút điều hướng phân trang -->
      <div class="pagination">
        <!-- Pagination for Phim Lẻ -->
        <button v-if="currentTab === 'movies'" @click="changePage(page - 1)" :disabled="page === 1">Trước</button>
        <span v-if="currentTab === 'movies'">Trang {{ page }} / {{ totalMoviePages }}</span>
        <button v-if="currentTab === 'movies'" @click="changePage(page + 1)"
          :disabled="page === totalMoviePages">Sau</button>

        <!-- Pagination for Phim Bộ -->
        <button v-if="currentTab === 'series'" @click="changePage(page - 1)" :disabled="page === 1">Trước</button>
        <span v-if="currentTab === 'series'">Trang {{ page }} / {{ totalSeriesPages }}</span>
        <button v-if="currentTab === 'series'" @click="changePage(page + 1)"
          :disabled="page === totalSeriesPages">Sau</button>
      </div>
    </div>
    <!-- Form Thêm Phim lẻ -->
    <div v-if="showSingleMovieForm" class="form-overlay">
      <div class="form-container">
        <h2>Thêm Phim Mới</h2>
        <form @submit.prevent="AddMovie" class="movie-form">
          <div class="form-columns">
            <div class="form-column left-column">
              <div class="form-group">
                <label>Tên phim</label>
                <input type="text" v-model="singleMovieForm.title" required />
              </div>
              <div class="form-group">
                <label>Diễn viên</label>
                <multiselect v-model="singleMovieForm.actors" :options="actorOptions" label="name" track-by="id"
                  placeholder="Chọn diễn viên" :multiple="true"></multiselect>
              </div>
              <div class="form-group">
                <label>Đạo diễn</label>
                <select v-model="singleMovieForm.directorId" required>
                  <option v-for="director in directorOptions" :value="director.id" :key="director.id">
                    {{ director.name }}
                  </option>
                </select>
              </div>

              <div class="form-group">
                <label>Năm Phát Hành</label>
                <input type="text" v-model="singleMovieForm.yearReleased" required />

              </div>
              <div class="form-group">
                <label>Mô tả</label>
                <input type="text" v-model="singleMovieForm.description" required />
              </div>
              <div class="form-group">
                <label>Hot</label>
                <select v-model="singleMovieForm.isHot" required>
                  <option value="true">True</option>
                  <option value="false">False</option>
                </select>
              </div>
            </div>

            <div class="form-column right-column">
              <div class="form-group">
                <label>Rating</label>
                <input v-model="singleMovieForm.rating" type="number" step="0.1" min="0.1" max="10" required />
              </div>
              <div class="form-group">
                <label>Thể Loại</label>
                <multiselect v-model="singleMovieForm.categories" :options="categorieOptions" label="name" track-by="id"
                  placeholder="Chọn thể loại" :multiple="true"></multiselect>
              </div>
              <div class="form-group">
                <label>Quốc gia</label>
                <input type="text" v-model="singleMovieForm.nation" required />
              </div>
              <div class="form-group">
                <label>Avatar</label>
                <input type="file" @change="onAvatarChangen" accept="image/*" />
                <img v-if="avatarPreview" :src="avatarPreview" alt="Avatar Preview"
                  style="width: 150px; height: auto; margin-top: 10px;" />
              </div>

              <div class="form-group">
                <label>Poster</label>
                <input type="file" @change="onPosterChangen" accept="image/*" />
                <img v-if="posterPreview" :src="posterPreview" alt="Poster Preview"
                  style="width: 150px; height: auto; margin-top: 10px;" />
              </div>
              <div class="form-group">
                <label>Link Film</label>
                <input v-model="singleMovieForm.linkFilmUrl" required placeholder="URL của phim" />
              </div>
            </div>
          </div>

          <div class="button-group">
            <button type="submit" class="submit-button">
              {{ singleMovieForm.update ? 'Cập nhật' : 'Thêm Phim' }}
            </button>
            <button type="button" class="cancel-button" @click="cancelSingleMovieForm">Hủy</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Form sửa phim lẻ -->
    <div v-if="showUpdateSingleMovieForm" class="form-overlay">
      <div class="form-container">
        <h2>Chỉnh sửa Phim</h2>
        <form @submit.prevent="UpdateMovie" class="movie-form">
          <div class="form-columns">
            <div class="form-column left-column">
              <div class="form-group">
                <label>ID</label>
                <input type="text" v-model="singleUpdateMovieForm.movieId" required />
              </div>
              <div class="form-group">
                <label>Tên phim</label>
                <input type="text" v-model="singleUpdateMovieForm.title" required />
              </div>
              <div class="form-group">
                <label>Diễn viên</label>
                <multiselect v-model="singleUpdateMovieForm.actors" :options="actorOptions" label="name" track-by="id"
                  placeholder="Chọn diễn viên" :multiple="true" :close-on-select="false" :preserve-search="true"
                  :show-labels="true"></multiselect>
              </div>
              <div class="form-group">
                <label>Đạo diễn</label>
                <select v-model="singleUpdateMovieForm.directorId" required>
                  <option v-for="director in directorOptions" :value="director.id" :key="director.id">
                    {{ director.name }}
                  </option>
                </select>
              </div>

              <div class="form-group">
                <label>Năm Phát Hành</label>
                <input type="text" v-model="singleUpdateMovieForm.yearReleased" required />
              </div>
              <div class="form-group">
                <label>Mô tả</label>
                <input type="text" v-model="singleUpdateMovieForm.description" required />
              </div>
              <div class="form-group">
                <label>Hot</label>
                <select v-model="singleUpdateMovieForm.isHot" required>
                  <option value="true">True</option>
                  <option value="false">False</option>
                </select>
              </div>
            </div>

            <div class="form-column right-column">
              <div class="form-group">
                <label>Rating</label>
                <input v-model="singleUpdateMovieForm.rating" type="number" step="0.1" min="0.1" max="10" required />
              </div>
              <div class="form-group">
                <label>Thể Loại</label>
                <multiselect v-model="singleUpdateMovieForm.categories" :options="categorieOptions" label="name"
                  track-by="id" placeholder="Chọn thể loại" :multiple="true"></multiselect>
              </div>
              <div class="form-group">
                <label>Quốc gia</label>
                <input type="text" v-model="singleUpdateMovieForm.nation" required />
              </div>
              <div class="form-group">
                <label>Avatar</label>
                <input type="file" @change="onAvatarChangee" accept="image/*" />
                <img v-if="avatarPreview" :src="avatarPreview" alt="Avatar Preview"
                  style="width: 150px; height: auto; margin-top: 10px;" />
              </div>

              <div class="form-group">
                <label>Poster</label>
                <input type="file" @change="onPosterChangee" accept="image/*" />
                <img v-if="posterPreview" :src="posterPreview" alt="Poster Preview"
                  style="width: 150px; height: auto; margin-top: 10px;" />
              </div>
              <div class="form-group">
                <label>Link Film</label>
                <input v-model="singleUpdateMovieForm.linkFilmUrl" required placeholder="URL của phim" />
              </div>
            </div>
          </div>

          <div class="button-group">
            <button type="submit" class="submit-button">
              {{ singleUpdateMovieForm.update ? 'Cập nhật' : 'Cập Nhật Phim' }}
            </button>
            <button type="button" class="cancel-button" @click="cancelUpdateSingleMovieForm">Hủy</button>
          </div>
        </form>
      </div>
    </div>
    <!-- Form Thêm Phim bộ -->
    <div v-if="showSeriesMovieForm" class="form-overlay">
      <div class="form-container">
        <h2>Thêm Phim Bộ</h2>
        <form @submit.prevent="AddSeries" class="movie-form">
          <div class="form-columns">
            <div class="form-column left-column">
              <div class="form-group">
                <label>Tên phim</label>
                <input type="text" v-model="seriesMovieForm.title" required />
              </div>
              <div class="form-group">
                <label>Phần</label>
                <input v-model="seriesMovieForm.season" type="number" step="1" min="1" required />
              </div>
              <div class="form-group">
                <label>Diễn viên</label>
                <multiselect v-model="seriesMovieForm.actors" :options="actorOptions" label="name" track-by="id"
                  placeholder="Chọn diễn viên" :multiple="true"></multiselect>
              </div>
              <div class="form-group">
                <label>Đạo diễn</label>
                <select v-model="seriesMovieForm.directorId" required>
                  <option v-for="director in directorOptions" :value="director.id" :key="director.id">
                    {{ director.name }}
                  </option>
                </select>
              </div>
              <div class="form-group">
                <label>Năm Phát Hành</label>
                <input type="text" v-model="seriesMovieForm.yearReleased" required />
              </div>
              <div class="form-group">
                <label>Hot</label>
                <select v-model="seriesMovieForm.isHot" required>
                  <option value="true">True</option>
                  <option value="false">False</option>
                </select>
              </div>
            </div>

            <div class="form-column right-column">
              <div class="form-group">
                <label>Rating</label>
                <input v-model="seriesMovieForm.rating" type="number" step="0.1" min="0.1" max="10" required />
              </div>
              <div class="form-group">
                <label>Thể Loại</label>
                <multiselect v-model="seriesMovieForm.categories" :options="categorieOptions" label="name" track-by="id"
                  placeholder="Chọn thể loại" :multiple="true"></multiselect>
              </div>
              <div class="form-group">
                <label>Quốc gia</label>
                <input type="text" v-model="seriesMovieForm.nation" required />
              </div>
              <div class="form-group">
                <label>Mô tả</label>
                <input type="text" v-model="seriesMovieForm.description" required />
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
                <img v-if="posterPreview" :src="posterPreview" alt="Poster Preview"
                  style="width: 150px; height: auto; margin-top: 10px;" />
              </div>
            </div>
          </div>

          <div class="button-group">
            <button type="submit" class="submit-button">
              {{ seriesMovieForm.update ? 'Cập nhật' : 'Thêm Phim' }}
            </button>
            <button type="button" class="cancel-button" @click="cancelSeriesMovieForm">Hủy</button>
          </div>
        </form>
      </div>
    </div>
    <!-- Form Sửa Phim bộ -->
    <div v-if="showUpdateSeriesMovieForm" class="form-overlay">
      <div class="form-container">
        <h2>Sửa Phim Bộ</h2>
        <form @submit.prevent="UpdateSeries" class="movie-form">
          <div class="form-columns">
            <div class="form-column left-column">
              <div class="form-group">
                <label>ID</label>
                <input type="text" v-model="seriesUpdateMovieForm.seriesId" required />
              </div>
              <div class="form-group">
                <label>Tên phim</label>
                <input type="text" v-model="seriesUpdateMovieForm.title" required />
              </div>
              <div class="form-group">
                <label>Phần</label>
                <input v-model="seriesUpdateMovieForm.season" type="number" step="1" min="1" required />
              </div>
              <div class="form-group">
                <label>Diễn viên</label>
                <multiselect v-model="seriesUpdateMovieForm.actors" :options="actorOptions" label="name" track-by="id"
                  placeholder="Chọn diễn viên" :multiple="true"></multiselect>
              </div>
              <div class="form-group">
                <label>Đạo diễn</label>
                <select v-model="seriesUpdateMovieForm.directorId" required>
                  <option v-for="director in directorOptions" :value="director.id" :key="director.id">
                    {{ director.name }}
                  </option>
                </select>
              </div>
              <div class="form-group">
                <label>Năm Phát Hành</label>
                <input type="text" v-model="seriesUpdateMovieForm.yearReleased" required />
              </div>
              <div class="form-group">
                <label>Hot</label>
                <select v-model="seriesUpdateMovieForm.isHot" required>
                  <option value="true">True</option>
                  <option value="false">False</option>
                </select>
              </div>
            </div>

            <div class="form-column right-column">
              <div class="form-group">
                <label>Rating</label>
                <input v-model="seriesUpdateMovieForm.rating" type="number" step="0.1" min="0.1" max="10" required />
              </div>
              <div class="form-group">
                <label>Thể Loại</label>
                <multiselect v-model="seriesUpdateMovieForm.categories" :options="categorieOptions" label="name"
                  track-by="id" placeholder="Chọn thể loại" :multiple="true"></multiselect>
              </div>
              <div class="form-group">
                <label>Quốc gia</label>
                <input type="text" v-model="seriesUpdateMovieForm.nation" required />
              </div>
              <div class="form-group">
                <label>Mô tả</label>
                <input type="text" v-model="seriesUpdateMovieForm.description" required />
              </div>
              <div class="form-group">
                <label>Avatar</label>
                <input type="file" @change="onAvatarChangenn" accept="image/*" />
                <img v-if="avatarPreview" :src="avatarPreview" alt="Avatar Preview"
                  style="width: 150px; height: auto; margin-top: 10px;" />
              </div>

              <div class="form-group">
                <label>Poster</label>
                <input type="file" @change="onPosterChangenn" accept="image/*" />
                <img v-if="posterPreview" :src="posterPreview" alt="Poster Preview"
                  style="width: 150px; height: auto; margin-top: 10px;" />
              </div>
            </div>
          </div>

          <div class="button-group">
            <button type="submit" class="submit-button">
              {{ seriesUpdateMovieForm.update ? 'Cập nhật' : 'Cập Nhật Phim' }}
            </button>
            <button type="button" class="cancel-button" @click="cancelUpdateSeriesMovieForm">Hủy</button>
          </div>
        </form>
      </div>
    </div>
    <!-- Modal Thêm Tập Phim -->
    <div v-if="showAddEpisodeModal" class="modal">
      <div class="modal-content">
        <h3>Thêm Tập Phim</h3>
        <label for="episodeNumber">Số tập:</label>
        <input v-model="newEpisode.episodeNumber" type="number" min="1" id="episodeNumber" required />

        <label for="linkFilmUrl">Link phim:</label>
        <input v-model="newEpisode.linkFilmUrl" id="linkFilmUrl" required />

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

        <label for="editLinkFilmUrl">Link phim:</label>
        <input v-model="editingEpisode.linkFilmUrl" id="editLinkFilmUrl" required />

        <div class="buttons-container">
          <button @click="updateEpisode">Cập nhật Tập Phim</button>
          <button @click="closeEditEpisodeModal">Hủy</button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import { ref } from "vue";
import Multiselect from "vue-multiselect";
import "vue-multiselect/dist/vue-multiselect.min.css";
const currentTab = ref("movies"); // Tab mặc định là "Phim Lẻ"
const paginatedMovies = ref([]); // Dữ liệu phim lẻ
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
      selectedSeriesId: null,
      editingEpisode: {},
      currentEpisodePage: 1,  // Trang hiện tại cho các tập phim
      episodesPerPage: 5,     // Số lượng tập phim hiển thị mỗi trang
      totalEpisodePages: 0,
      showSingleMovieForm: false,
      showUpdateSingleMovieForm: false,
      showSeriesMovieForm: false,
      showUpdateSeriesMovieForm: false,
      actorOptions: [], // Danh sách diễn viên từ API
      categorieOptions: [],
      directorOptions: '',
      searchQuery: "",
      allMovies: [],
      allSeries: [],
      loading: false,
      error: null,
      avatarPreview: null,
      posterPreview: null,
      currentTab: "movies",
      movieList: [],
      seriesList: [],
      page: 1, // Trang hiện tại
      itemsPerPage: 5, // 5 phim trên mỗi trang

      singleMovieForm: {
        title: "",
        directorId: "",
        yearReleased: "",
        nation: "",
        actors: [],
        categories: [],
        directors: '',
        rating: "",
        editing: false,
        update: false,
        isHot: "false",
        linkFilm: "",
        posterFile: null,
        avatarFile: null,
        description: "",
      },
      singleUpdateMovieForm: {
        title: "",
        director: "",
        yearReleased: "",
        nation: "",
        actors: [],
        categories: [],
        directors: '',
        rating: "",
        update: false,
        isHot: "false",
        linkFilm: "",
        posterFile: null,
        avatarFile: null,
        description: "",
      },
      seriesMovieForm: {
        title: "",
        season: "",
        totalEpisode: "",
        nation: "",
        actors: [],
        directors: "",
        yearReleased: "",
        categories: [],
        description: "",
        isHot: false,
        linkFilm: "",
        posterFile: null,
        ratinrFile: null,
        avatarFile: null,
        editing: false,
        update: false,
      },
      seriesUpdateMovieForm: {
        title: "",
        season: "",
        totalEpisode: "",
        nation: "",
        actors: [],
        directors: "",
        yearReleased: "",
        categories: [],
        description: "",
        isHot: false,
        update: false,
        linkFilm: "",
        posterFile: null,
        avatarFile: null,
        rating: null,
      },
    };
  },
  computed: {
    // Tổng số trang phim
    totalMoviePages() {
      return Math.ceil(this.movieList.length / this.itemsPerPage);
    },
    totalSeriesPages() {
      return Math.ceil(this.seriesList.length / this.itemsPerPage);
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
    filteredMovies() {
      const query = this.searchQuery.trim().toLowerCase();
      return this.movieList.filter(
        (movie) =>
          movie.title.toLowerCase().includes(query) ||
          movie.description.toLowerCase().includes(query)
      );
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
    async fetchActorOptions() {
      // Gọi API để lấy danh sách diễn viên
      try {
        const response = await axios.get("http://localhost:5148/api/AdminActor", {
          params: {
            search: "",
            sortBy: "ActorId",
            sortDirection: "asc",
            page: 1,
            pageSize: 70, // Hiển thị tất cả diễn viên trong một lần tải
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

    watch: {
      showSingleMovieForm(newVal) {
        if (newVal) {
          this.fetchActorOptions();
          this.fetchcategorieOptions();// Tải danh sách diễn viên khi hiển thị form
          this.fetchdirectorOptions();
        }
      },
    },
    watch: {
      showUpdateSingleMovieForm(newVal) {
        if (newVal) {
          this.fetchActorOptions();
          this.fetchcategorieOptions();
          this.fetchdirectorOptions();
        }
      },
    },

    watch: {
      showSeriesMovieForm(newVal) {
        if (newVal) {
          this.fetchActorOptions();
          this.fetchcategorieOptions();// Tải danh sách diễn viên khi hiển thị form
          this.fetchdirectorOptions();
        }
      },
    },
    watch: {
      showUpdateSeriesMovieForm(newVal) {
        if (newVal) {
          this.fetchActorOptions(); // Tải danh sách diễn viên khi hiển thị form
        }
      },
    },
    async fetchcategorieOptions() {
      // Gọi API để lấy danh sách thể loại
      try {
        const response = await axios.get("http://localhost:5148/api/AdminCategories", {
          params: {
            search: "",
            sortBy: "CategoryId",
            sortDirection: "asc",
            page: 1,
            pageSize: 70, // Hiển thị tất cả diễn viên trong một lần tải
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
        const response = await axios.get("http://localhost:5148/api/AdminDirectors/List-Directors", {
          params: {
            search: "",
            sortBy: "DirectorId",
            sortDirection: "asc",
            page: 1,
            pageSize: 70, // Hiển thị tất cả diễn viên trong một lần tải
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

    // Movie
    onAvatarChangen(event) {
      const file = event.target.files[0];
      this.avatarFile = file;
      this.avatarPreview = URL.createObjectURL(file);
      this.singleMovieForm.avatarFile = file;
    },
    onPosterChangen(event) {
      const file = event.target.files[0];
      this.posterFile = file;
      this.posterPreview = URL.createObjectURL(file);
      this.singleMovieForm.posterFile = file;
    },
    onAvatarChangee(event) {
      const file = event.target.files[0];
      if (file) {
        this.avatarPreview = URL.createObjectURL(file);
        this.singleUpdateMovieForm.avatarFile = file; // Lưu file vào form
      } else {
        this.avatarPreview = null;
      }
    },

    onPosterChangee(event) {
      const file = event.target.files[0];
      if (file) {
        this.posterPreview = URL.createObjectURL(file);
        this.singleUpdateMovieForm.posterFile = file; // Lưu file vào form
      } else {
        this.posterPreview = null;
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
    onAvatarChangenn(event) {
      const file = event.target.files[0];
      if (file) {
        this.avatarPreview = URL.createObjectURL(file);
        this.seriesUpdateMovieForm.avatarFile = event.target.files[0];
      } else {
        this.avatarPreview = null;
      }
    },
    onPosterChangenn(event) {
      console.log(event.target.files);
      const file = event.target.files[0];
      if (file) {
        this.posterPreview = URL.createObjectURL(file);
        this.seriesUpdateMovieForm.posterFile = event.target.files[0];
      } else {
        this.posterPreview = null;
      }
    },

    async AddMovie() {
      try {
        const {
          title,
          description,
          directorId,
          nation,
          rating,
          posterFile,
          avatarFile,
          isHot,
          yearReleased,
          actors,
          categories,
        } = this.singleMovieForm;

        console.log('Dữ liệu biểu mẫu:', this.singleMovieForm);

        // Kiểm tra các trường bắt buộc
        if (
          !title ||
          !description ||
          !directorId ||
          !nation ||
          !rating ||
          !posterFile ||
          !avatarFile ||
          !yearReleased
        ) {
          alert("Vui lòng nhập đầy đủ thông tin!");
          return;
        }

        // Chuẩn bị FormData
        const formData = new FormData();
        formData.append("Title", title);
        formData.append("Description", description);
        formData.append("DirectorId", directorId);
        formData.append("Nation", nation);
        formData.append("Rating", rating);
        formData.append("posterFile", posterFile); // Đồng nhất với API
        formData.append("AvatarUrlFile", avatarFile); // Đồng nhất với API
        formData.append("IsHot", isHot ? "true" : "false");
        formData.append("YearReleased", yearReleased);
        formData.append("ActorIds", actors.map((actor) => actor.id).join(",")); // Đúng định dạng
        formData.append("CategoryIds", categories.map((category) => category.id).join(",")); // Đúng định dạng
        // Xác nhận trước khi gửi
        if (!confirm("Bạn có chắc chắn muốn thêm phim bộ này không?")) {
          return;
        }

        // Thực hiện yêu cầu API
        const response = await axios.post(
          "http://localhost:5148/api/AdminMovies/AddMovie",
          formData,
          {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          }
        );

        // Xử lý phản hồi
        alert("Thêm phim bộ thành công!");
        this.fetchMovies(); // Làm mới danh sách phim
        this.movieList.unshift(response.data); // Cập nhật danh sách phim
        this.updateMoviePagination(); // Cập nhật phân trang
        this.showSingleMovieForm = false; // Ẩn biểu mẫu khi thêm thành công
      } catch (error) {
        console.error(
          "Lỗi khi thêm phim bộ:",
          error.response?.data?.message || error.message
        );
        alert(`Đã xảy ra lỗi: ${error.response?.data?.message || error.message}`);
      }
    },

    async UpdateMovie() {
      try {
        // Lấy dữ liệu từ form
        const {
          movieId,
          title,
          description,
          directorId,
          nation,
          yearReleased,
          rating,
          posterFile,
          avatarFile,
          actors,
          isHot,
          linkFilmUrl,
          categories,
        } = this.singleUpdateMovieForm;

        // Kiểm tra các trường bắt buộc
        if (!title) {
          alert("Vui lòng nhập tiêu đề phim!");
          return;
        }

        // Chuẩn bị FormData
        const formData = new FormData();
        formData.append("ID", movieId);
        formData.append("Title", title);
        formData.append("Description", description);
        formData.append("DirectorId", directorId);
        formData.append("Nation", nation);
        formData.append("YearReleased", yearReleased);
        formData.append("Rating", rating);
        formData.append("IsHot", isHot ? "true" : "false");
        formData.append("ActorIds", actors.map((actor) => actor.id).join(","));
        formData.append("CategoryIds", categories.map((cat) => cat.id).join(","));
        formData.append("LinkFilmUrl", linkFilmUrl || "");
        formData.append("posterFile", posterFile); // Đồng nhất với API
        formData.append("AvatarUrlFile", avatarFile);

        // Xác nhận trước khi gửi
        if (!confirm("Bạn có chắc chắn muốn cập nhật phim này không?")) {
          return;
        }

        // Gửi yêu cầu tới API
        const response = await axios.put(
          `http://localhost:5148/api/AdminMovies/UpdateMovie/${movieId}`,
          formData,
          {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          }
        );

        // Hiển thị thông báo thành công
        alert("Cập nhật phim thành công!");
        this.fetchMovies(); // Làm mới danh sách phim
        this.showUpdateSingleMovieForm = false; // Ẩn form chỉnh sửa

      } catch (error) {
        // Xử lý lỗi và log chi tiết
        console.error("Lỗi khi cập nhật phim:", error.response?.data?.message || error.message);
        alert(`Đã xảy ra lỗi: ${error.response?.data?.message || error.message}`);
      }
    },

    async fetchMovies() {
      this.loading = true;
      this.error = null;
      try {
        const response = await axios.get("http://localhost:5148/api/AdminMovies", {
          params: {

            search: this.searchQuery.trim()
          },
        });
        this.movieList = (Array.isArray(response.data) ? response.data : []).sort(
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
    async deleteMovie(index) {
      if (confirm("Bạn có chắc muốn xóa phim này?")) {
        try {
          // Tính chỉ số thực trong danh sách gốc
          const globalIndex = (this.page - 1) * this.itemsPerPage + index;

          // Lấy ID của phim cần xóa
          const movieId = this.allMovies[globalIndex].movieId;

          // Gửi yêu cầu xóa phim tới server
          await axios.delete(`http://localhost:5148/api/AdminMovies/de/${movieId}`);

          // Làm mới danh sách phim
          this.fetchMovies();
        } catch (error) {
          console.error("Lỗi khi xóa phim:", error);
        }
      }
    },
    async fetchSeries() {
      this.loading = true;
      this.error = null;
      try {
        const response = await axios.get("http://localhost:5148/api/AdminSeries", {
          params: {
            search: this.searchQuery.trim()
          },
        });
        // Khởi tạo sẵn showEpisodes và episodes cho từng series
        this.seriesList = (Array.isArray(response.data) ? response.data : [])
          .map(s => ({
            ...s,
            showEpisodes: false,
            episodes: []
          }))
          .sort();

        this.allSeries = Array.isArray(response.data) ? response.data : [];
        this.totalSeriesPages = Math.ceil(this.allSeries.length / this.itemsPerPage);
        this.updateSeriesPagination();
      } catch (error) {
        console.error("Lỗi khi tải danh sách phim:", error.response?.data?.message || error.message);
        this.error = "Không thể tải danh sách phim!";
      } finally {
        this.loading = false;
      }
    },
    updateSeriesPagination() {
      const startIndex = (this.page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      this.paginatedSeries = this.seriesList.slice(startIndex, endIndex);
    },
    // Xoá phim bộ
    async deleteSeries(index) {
      try {
        // Xác nhận trước khi xóa
        if (!confirm("Bạn có chắc chắn muốn xóa phim này?")) {
          return; // Người dùng hủy thao tác xóa
        }

        // Tính chỉ số thực trong danh sách gốc
        const globalIndex = (this.page - 1) * this.itemsPerPage + index;

        // Kiểm tra tính hợp lệ của chỉ số và dữ liệu
        const series = this.allSeries[globalIndex];
        if (!series || !series.seriesId) {
          alert("Không thể xác định phim để xóa. Vui lòng thử lại!");
          return;
        }

        // Lấy ID của phim cần xóa
        const seriesId = series.seriesId;

        // Gửi yêu cầu xóa phim tới server
        const response = await axios.delete(
          `http://localhost:5148/api/AdminSeries/soft-delete/${seriesId}`,
          {
            headers: {
              accept: "*/*", // Header theo yêu cầu của API
            },
          }
        );

        // Hiển thị thông báo thành công
        alert(response.data.message || "Phim đã được xóa mềm thành công!");
        console.log("Kết quả xóa mềm:", response.data);
        this.fetchSeries();
        // Loại bỏ phim đã xóa khỏi danh sách hiển thị
        this.allSeries.splice(globalIndex, 1); // Xóa khỏi danh sách gốc
        this.updateSeriesPagination(); // Cập nhật lại danh sách phân trang
      } catch (error) {
        // Xử lý lỗi chi tiết
        console.error("Lỗi khi xóa phim:", error.response?.data || error.message);
        alert(`Đã xảy ra lỗi: ${error.response?.data?.message || error.message}`);
      }
    },
    async toggleEpisodes(series) {
      if (!series.showEpisodes) {
        try {
          // Gọi API để lấy danh sách tập phim
          const response = await axios.get(
            `http://localhost:5148/api/AdminEpisode/BySeries/${series.seriesId}?pageNumber=1&pageSize=100`
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
        linkFilmUrl: ''
      };
      this.showAddEpisodeModal = true;
    },

    async addEpisode() {
      try {
        const formData = new FormData();
        formData.append("SeriesId", this.selectedSeriesId);
        formData.append("EpisodeNumber", this.newEpisode.episodeNumber);
        formData.append("LinkFilmUrl", this.newEpisode.linkFilmUrl);

        const response = await axios.post(
          "http://localhost:5148/api/AdminEpisode/AddEpisode",
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
        linkFilmUrl: ''
      };
    },

    openEditEpisodeModal(episode) {
      this.editingEpisode = { ...episode }; // Sao chép dữ liệu tập phim để chỉnh sửa
      this.showEditEpisodeModal = true;
    },

    async updateEpisode() {
      try {
        // Tạo đối tượng FormData
        const formData = new FormData();
        formData.append('newLink', this.editingEpisode.linkFilmUrl); // Gửi link phim mới

        // Kiểm tra xem seriesId và episodeNumber có hợp lệ không
        if (!this.editingEpisode.seriesId || !this.editingEpisode.episodeNumber) {
          alert("Vui lòng chọn một series và số tập hợp lệ.");
          return; // Dừng lại nếu không hợp lệ
        }

        // Gửi yêu cầu PUT với dữ liệu FormData
        const response = await axios.put(
          `http://localhost:5148/api/AdminEpisode/UpdateEpisode?seriesId=${this.editingEpisode.seriesId}&episodeNumber=${this.editingEpisode.episodeNumber}`,
          formData,
          {
            headers: { "Content-Type": "multipart/form-data" } // Đặt header đúng
          }
        );

        // Kiểm tra phản hồi từ server
        if (response.status === 200) {
          // Hiển thị thông báo sửa thành công
          if (this.$toast) {
            this.$toast.success("Tập phim đã được sửa thành công!");
          }

          // Cập nhật danh sách tập phim
          const series = this.seriesList.find(s => s.seriesId === this.editingEpisode.seriesId);
          if (series) {
            const updatedEpisodeIndex = series.episodes.findIndex(episode => episode.episodeNumber === this.editingEpisode.episodeNumber);
            if (updatedEpisodeIndex !== -1) {
              // Cập nhật thông tin tập phim trong danh sách
              series.episodes[updatedEpisodeIndex].linkFilmUrl = this.editingEpisode.linkFilmUrl;
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
        axios.delete(`http://localhost:5148/api/AdminEpisode/DeleteEpisode/${episodeId}`)
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

    // Thêm Phim Bộ
    async AddSeries() {
      try {
        const {
          title,
          description,
          directorId,
          nation,
          rating,
          posterFile,
          avatarFile,
          isHot,
          yearReleased,
          actors,
          categories,
          season,
        } = this.seriesMovieForm;

        console.log('Dữ liệu biểu mẫu:', this.seriesMovieForm);

        // Kiểm tra các trường bắt buộc
        if (
          !title ||
          !description ||
          !directorId ||
          !nation ||
          !rating ||
          !posterFile ||
          !avatarFile ||
          !yearReleased
        ) {
          alert("Vui lòng nhập đầy đủ thông tin!");
          return;
        }

        // Chuẩn bị FormData
        const formData = new FormData();
        formData.append("Title", title);
        formData.append("Description", description);
        formData.append("DirectorId", directorId);
        formData.append("Nation", nation);
        formData.append("Rating", rating);
        formData.append("posterFile", posterFile); // Đồng nhất với API
        formData.append("AvatarUrlFile", avatarFile); // Đồng nhất với API
        formData.append("IsHot", isHot ? "true" : "false");
        formData.append("YearReleased", yearReleased);
        formData.append("ActorIds", actors.map((actor) => actor.id).join(",")); // Đúng định dạng
        formData.append("CategoryIds", categories.map((category) => category.id).join(",")); // Đúng định dạng
        formData.append("Season", season);

        // Xác nhận trước khi gửi
        if (!confirm("Bạn có chắc chắn muốn thêm phim bộ này không?")) {
          return;
        }

        // Thực hiện yêu cầu API
        const response = await axios.post(
          "http://localhost:5148/api/AdminSeries/AddSeries",
          formData,
          {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          }
        );

        // Xử lý phản hồi
        alert("Thêm phim bộ thành công!");
        this.fetchSeries(); // Làm mới danh sách phim
        this.seriesList.unshift(response.data); // Cập nhật danh sách phim
        this.updateSeriesPagination(); // Cập nhật phân trang
        this.showSeriesMovieForm = false; // Ẩn biểu mẫu khi thêm thành công
      } catch (error) {
        console.error(
          "Lỗi khi thêm phim bộ:",
          error.response?.data?.message || error.message
        );
        alert(`Đã xảy ra lỗi: ${error.response?.data?.message || error.message}`);
      }
    },
    // Sửa Phim Bộ 
    async UpdateSeries() {
      try {
        // Lấy dữ liệu từ form
        const {
          seriesId,
          title,
          description,
          directorId,
          nation,
          rating,
          posterFile,
          avatarFile,
          isHot,
          yearReleased,
          actors,
          categories,
          linkFilm,
        } = this.seriesUpdateMovieForm;

        console.log("Dữ liệu biểu mẫu:", this.seriesUpdateMovieForm);

        // Kiểm tra các trường bắt buộc
        if (
          !seriesId ||
          !title ||
          !description ||
          !directorId ||
          !nation ||
          !rating ||
          !yearReleased
        ) {
          alert("Vui lòng nhập đầy đủ thông tin bắt buộc!");
          return;
        }

        // Chuẩn bị FormData
        const formData = new FormData();
        formData.append("Title", title);
        formData.append("Description", description);
        formData.append("DirectorId", directorId);
        formData.append("Nation", nation);
        formData.append("Rating", rating);
        if (posterFile) formData.append("posterFile", posterFile); // File poster (nếu có)
        if (avatarFile) formData.append("AvatarUrlFile", avatarFile); // File avatar (nếu có)
        formData.append("IsHot", isHot ? "true" : "false");
        formData.append("YearReleased", yearReleased);
        formData.append("ActorIds", actors.map((actor) => actor.id).join(",")); // Danh sách ID diễn viên
        formData.append("CategoryIds", categories.map((category) => category.id).join(",")); // Danh sách ID thể loại
        formData.append("LinkFilmUrl", linkFilm || ""); // Link phim (không bắt buộc)

        // Gửi yêu cầu cập nhật API
        const response = await axios.put(
          `http://localhost:5148/api/AdminSeries/UpdateSeries/${seriesId}`,
          formData,
          {
            headers: {
              "Content-Type": "multipart/form-data",
            },
          }
        );

        // Hiển thị thông báo thành công
        alert("Cập nhật phim thành công!");
        this.fetchSeries(); // Làm mới danh sách phim
        this.showUpdateSeriesMovieForm = false; // Ẩn form chỉnh sửa

      } catch (error) {
        // Xử lý lỗi và log chi tiết
        console.error("Lỗi khi cập nhật phim:", error.response?.data?.message || error.message);
        alert(`Đã xảy ra lỗi: ${error.response?.data?.message || error.message}`);
      }
    },

    async searchSeries() {
      this.page = 1; // Đặt lại về trang đầu tiên
      await this.fetchSeries(); // Gọi lại API với từ khóa tìm kiếm
    },
    selectTab(tab) {
      this.currentTab = tab;
      this.searchQuery = ""; // Xóa từ khóa tìm kiếm khi chuyển tab
      this.page = 1; // Đặt lại về trang đầu tiên
      if (tab === "movies") {
        this.fetchMovies();
      } else if (tab === "series") {
        this.fetchSeries();
      }
    },
    // Thay đổi trang
    changePage(newPage) {
      if (newPage >= 1 && newPage <= this.totalMoviePages) {
        this.page = newPage; // Cập nhật trang hiện tại
      }
    },

    searchMovies() {
      this.fetchMovies(); // Tải phim dựa vào API (hỗ trợ tìm kiếm nếu API được cấu hình)
    },
    searchSeries() {
      this.fetchSeries(); // Tải phim bộ dựa vào API (hỗ trợ tìm kiếm nếu API được cấu hình)
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

    editMovie(index) {
      const globalIndex = (this.page - 1) * this.itemsPerPage + index;
      const movie = this.allMovies[globalIndex];

      // Chuyển đổi dữ liệu đúng định dạng
      this.singleUpdateMovieForm = {
        movieId: movie.movieId,
        title: movie.title || "",
        description: movie.description || "",
        directorId: movie.directorId || null,
        nation: movie.nation || "",
        yearReleased: movie.yearReleased || "",
        rating: movie.rating || "",
        isHot: movie.isHot || false,
        linkFilmUrl: movie.linkFilmUrl || "",
        posterFile: null, // Người dùng cần upload lại
        avatarFile: null,

        // Chuyển actor từ API về multiselect [{id, name}]
        actors: movie.actors?.map(actor => {
  const matched = this.actorOptions.find(a => a.id === actor.id || a.id === actor.actorId);
  return matched ? matched : { id: actor.id, name: actor.name || 'Không rõ' };
}) || [],

        // Chuyển categories từ API về multiselect [{id, name}]
        categories: movie.categories?.map(cat => {
  const matched = this.categorieOptions.find(c => c.id === cat.id || c.id === cat.categoryId);
  return matched ? matched : { id: cat.id, name: cat.name || 'Không rõ' };
}) || [],
      };
      // Kiểm tra giá trị của actors và categories
      console.log('Actors:', this.singleUpdateMovieForm.actors);
      console.log('Categories:', this.singleUpdateMovieForm.categories);
      // Cập nhật preview ảnh và poster
      this.avatarPreview = movie.avatarUrl || null; // Giả sử movie.avatarUrl chứa đường dẫn đến ảnh đại diện
      this.posterPreview = movie.posterUrl || null; // Giả sử movie.posterUrl chứa đường dẫn đến poster
      this.showUpdateSingleMovieForm = true;
    },


    editSeries(index) {
      // Tính chỉ số toàn cục dựa trên phân trang
      const globalIndex = (this.page - 1) * this.itemsPerPage + index;
      const series = this.allSeries[globalIndex]; // Lấy đối tượng series từ danh sách gốc
      // Chuyển đổi dữ liệu để đưa vào form chỉnh sửa
      this.seriesUpdateMovieForm = {
        seriesId: series.seriesId, // Lấy seriesId từ đối tượng series
        title: series.title || "",
        description: series.description || "",
        directorId: series.directorId || null,
        nation: series.nation || "",
        yearReleased: series.yearReleased || "",
        rating: series.rating || "",
        isHot: series.isHot || false,
        linkFilmUrl: series.linkFilmUrl || "",
        season: series.season || "",
        posterFile: null, // Người dùng cần tải lại poster
        avatarFile: null, // Người dùng cần tải lại avatar

        // Chuyển actor từ API về multiselect [{id, name}]
        actors: series.actors?.map(actor => ({
          id: actor.id,
          name: actor.name,
        })) || [],

        // Chuyển categories từ API về multiselect [{id, name}]
        categories: series.categories?.map(cat => ({
          id: cat.id,
          name: cat.name,
        })) || [],
      };
      // Cập nhật preview ảnh và poster
      this.avatarPreview = series.avatarUrl || null; // Giả sử movie.avatarUrl chứa đường dẫn đến ảnh đại diện
      this.posterPreview = series.posterUrl || null; // Giả sử movie.posterUrl chứa đường dẫn đến poster
      // Hiển thị form chỉnh sửa
      this.showUpdateSeriesMovieForm = true;
    },

    addActor() {
      this.singleMovieForm.actors.push({ nameAct: "" });
      this.seriesMovieForm.actors.push({ nameAct: "" });
    },
    addActor() {
      this.singleUpdateMovieForm.actors.push({ nameAct: "" });
      this.seriesUpdateMovieForm.actors.push({ nameAct: "" });

    },
    cancelSingleMovieForm() {
      this.singleMovieForm = {
        title: "",
        director: "",
        yearReleased: "",
        nation: "",
        actors: [],
        categories: [],
        directors: '',
        rating: "",
        status: "Công chiếu",
        editing: false,
        update: false,
        isHot: "false", // Default value for Hot
        linkFilm: "", // New field for film link
        posterFile: null, // File poster
        avatarFile: null, // File đại diện
        description: "",
      }; // Reset dữ liệu
      this.showSingleMovieForm = false;
    },
    cancelUpdateSingleMovieForm() {
      this.singleUpdateMovieForm = {
        title: "",
        directorId: "",
        yearReleased: "",
        nation: "",
        actors: [],
        categories: [],
        directors: '',
        rating: "",
        status: "Công chiếu",
        editing: false,
        update: false,
        isHot: "false", // Default value for Hot
        linkFilm: "", // New field for film link
        posterFile: null, // File poster
        avatarFile: null, // File đại diện
        description: "",
      }; // Reset dữ liệu
      this.showUpdateSingleMovieForm = false;
    },
    cancelSeriesMovieForm() {
      this.seriesMovieForm = {
        title: '',
        season: null,
        totalEpisode: null,
        nation: "",
        actors: [],
        directors: '',
        yearReleased: '',
        categories: [],
        description: '',
        isHot: false,
        linkFilm: "", // New field for film link
        posterFile: null, // File poster
        avatarFile: null, // File đại diện
        rating: null,
        status: '',
        editing: false,
        update: false,
      }
      this.showSeriesMovieForm = false;
    },
    cancelUpdateSeriesMovieForm() {
      this.seriesUpdateMovieForm = {
        title: '',
        season: null,
        totalEpisode: null,
        nation: "",
        actors: [],
        directors: '',
        yearReleased: '',
        categories: [],
        description: '',
        isHot: false,
        linkFilm: "", // New field for film link
        posterFile: null, // File poster
        avatarFile: null, // File đại diện
        rating: null,
        status: '',
        editing: false,
        update: false,
      }
      this.showUpdateSeriesMovieForm = false;
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
.view-button,
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

.view-button {
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
