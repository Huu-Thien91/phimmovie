<template>
  <div class="container">
    <div class="movie-management">
      <h1>🎥 Quản lý Phim</h1>

      <!-- Tabs -->
      <div class="tabs">
        <button @click="selectTab('movies')" :class="{ active: currentTab === 'movies' }">Phim Lẻ</button>
      </div>

      <!-- Danh sách phim -->
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
              <th>Trạng thái</th>
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
              <td>{{ movie.status }}</td>
              <td>
                <button @click="editMovie(index)" class="edit-button">Sửa</button>
                <button @click="deleteMovie(index)" class="delete-button">Xóa</button>
              </td>
            </tr>
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
                <label>Trạng thái</label>
                <select v-model="singleMovieForm.status" required>
                  <option value="Công chiếu">Công chiếu</option>
                  <option value="Sắp ra mắt">Sắp ra mắt</option>
                </select>
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
                <input type="url" v-model="singleMovieForm.linkFilmUrl" required placeholder="URL của phim" />
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
                  placeholder="Chọn diễn viên" :multiple="true"></multiselect>
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
                <label>Trạng thái</label>
                <select v-model="singleUpdateMovieForm.status" required>
                  <option value="Công chiếu">Công chiếu</option>
                  <option value="Sắp ra mắt">Sắp ra mắt</option>
                </select>
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
                <input type="file" @change="onPosterChange" accept="image/*" />
              </div>
              <div class="form-group">
                <label>Link Film</label>
                <input type="url" v-model="singleUpdateMovieForm.linkFilmUrl" required placeholder="URL của phim" />
              </div>
            </div>
          </div>

          <div class="button-group">
            <button type="submit" class="submit-button">
              {{ singleUpdateMovieForm.update ? 'Cập nhật' : 'Thêm Phim' }}
            </button>
            <button type="button" class="cancel-button" @click="cancelUpdateSingleMovieForm">Hủy</button>
          </div>
        </form>
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
const selectTab = (tab) => {
  currentTab.value = tab; // Chuyển đổi tab
};

export default {
  components: {
    Multiselect,
  },
  data() {
    return {
      showSingleMovieForm: false,
      showUpdateSingleMovieForm: false,
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
      paginatedMovies: [],
      totalMoviePages: 0,
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
        status: "Công chiếu",
        editing: false,
        update: false,
        isHot: "false", // Default value for Hot
        linkFilm: "", // New field for film link
        posterFile: null, // File poster
        avatarFile: null, // File đại diện
        description: "", // Mô tả phim
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
        status: "Công chiếu",
        update: false,
        isHot: "false", // Default value for Hot
        linkFilm: "", // New field for film link
        posterFile: null, // File poster
        avatarFile: null, // File đại diện
        description: "", // Mô tả phim
      },
    };
  },
  computed: {
    // Tổng số trang phim
    totalMoviePages() {
      return Math.ceil(this.movieList.length / this.itemsPerPage);
    },
    paginatedMovies() {
      const start = (this.page - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.movieList.slice(start, end);
    },
    filteredMovies() {
      const query = this.searchQuery.trim().toLowerCase();
      return this.movieList.filter(
        (movie) =>
          movie.title.toLowerCase().includes(query) ||
          movie.description.toLowerCase().includes(query)
      );
    },
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

    watch: {
      showSingleMovieForm(newVal) {
        if (newVal) {
          this.fetchActorOptions(); // Tải danh sách diễn viên khi hiển thị form
        }
      },
    },
    watch: {
      showUpdateSingleMovieForm(newVal) {
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
    watch: {
      showSingleMovieForm(newVal) {
        if (newVal) {
          console.log(2)
          this.fetchcategorieOptions(); // Tải danh sách diễn viên khi hiển thị form
        }
      },
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
          console.log(1)
          this.fetchdirectorOptions(); // Tải danh sách diễn viên khi hiển thị form
        }
      },
    },

    // Movie
    onAvatarChange(event) {
      const file = event.target.files[0];
      if (file) {
        this.avatarPreview = URL.createObjectURL(file);
        this.singleMovieForm.avatarFile = event.target.files[0];
      } else {
        this.avatarPreview = null;
      }
    },
    onPosterChange(event) {
      console.log(event.target.files);
      const file = event.target.files[0];
      if (file) {
        this.posterPreview = URL.createObjectURL(file);
        this.singleMovieForm.posterFile = event.target.files[0];
      } else {
        this.posterPreview = null;
      }
    },
    onAvatarChangee(event) {
      const file = event.target.files[0];
      if (file) {
        this.avatarPreview = URL.createObjectURL(file);
        this.singleUpdateMovieForm.avatarFile = event.target.files[0];
      } else {
        this.avatarPreview = null;
      }
    },
    onPosterChangee(event) {
      console.log(event.target.files);
      const file = event.target.files[0];
      if (file) {
        this.posterPreview = URL.createObjectURL(file);
        this.singleUpdateMovieForm.posterFile = event.target.files[0];
      } else {
        this.posterPreview = null;
      }
    },
    async AddMovie() {
  const formData = new FormData();

  // Các trường cơ bản
  formData.append("Title", this.singleMovieForm.title);
  formData.append("Description", this.singleMovieForm.description);
  formData.append("Rating", this.singleMovieForm.rating);
  formData.append("DirectorId", this.singleMovieForm.directorId);
  formData.append("IsHot", this.singleMovieForm.isHot === 'true' || this.singleMovieForm.isHot === true ? 'true' : 'false');
  formData.append("YearReleased", this.singleMovieForm.yearReleased);
  formData.append("Nation", this.singleMovieForm.nation);
  formData.append("LinkFilmUrl", this.singleMovieForm.linkFilmUrl);

  // File ảnh avatar và poster
  if (this.singleMovieForm.avatar) {
    formData.append("AvatarUrlFile", this.singleMovieForm.avatar);
  }
  if (this.singleMovieForm.poster) {
    formData.append("posterFile", this.singleMovieForm.poster);
  }

  // ID của category và actor (nếu là nhiều thì dùng append lặp lại)
  if (this.singleMovieForm.categories) {
    this.singleMovieForm.categories.forEach(c => {
      formData.append("CategoryIds", c.id);
    });
  }

  if (this.singleMovieForm.actors) {
    this.singleMovieForm.actors.forEach(a => {
      formData.append("ActorIds", a.id);
    });
  }

  try {
    const response = await axios.post("http://localhost:5148/api/AdminMovie/AddMovie", formData, {
      headers: {
        "Content-Type": "multipart/form-data",
      }
    });

    alert("Thêm phim thành công!");
    this.cancelSingleMovieForm();
    this.getAllMovies(); // load lại danh sách nếu có
  } catch (error) {
    console.error("Lỗi khi thêm phim:", error);
    alert("Lỗi khi thêm phim!");
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
          posterFile,     // File poster (có thể là undefined nếu không thay đổi)
          avatarFile,     // File avatar (có thể là undefined nếu không thay đổi)
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
        formData.append("Description", description || "");
        formData.append("DirectorId", directorId);
        formData.append("Nation", nation);
        formData.append("YearReleased", yearReleased);
        formData.append("Rating", rating);
        formData.append("IsHot", isHot ? "true" : "false");
        formData.append("ActorIds", actors.map((actor) => actor.id).join(","));
        formData.append("CategoryIds", categories.map((cat) => cat.id).join(","));
        formData.append("LinkFilmUrl", linkFilmUrl || "");

        // Chỉ thêm file nếu người dùng chọn file mới
        if (posterFile) {
          formData.append("posterFile", posterFile);
        }
        if (avatarFile) {
          formData.append("AvatarUrlFile", avatarFile);
        }

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
        const response = await axios.get("http://localhost:5148/api/AdminMovie", {
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
        this.updatePagination();
      } catch (error) {
        console.error("Lỗi khi tải danh sách phim:", error.response?.data?.message || error.message);
        this.error = "Không thể tải danh sách phim!";
      } finally {
        this.loading = false;
      }
    },
    updatePagination() {
      const startIndex = (this.page - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;
      this.paginatedMovies = this.movieList.slice(startIndex, endIndex);
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
        actors: movie.actors?.map(actor => ({
          id: actor.id,
          name: actor.name,
        })) || [],

        // Chuyển categories từ API về multiselect [{id, name}]
        categories: movie.categories?.map(cat => ({
          id: cat.id,
          name: cat.name,
        })) || [],
      };

      this.showUpdateSingleMovieForm = true;
    },

    addActor() {
      this.singleMovieForm.actors.push({ nameAct: "" });
    },
    addActor() {
      this.singleUpdateMovieForm.actors.push({ nameAct: "" });

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
      this.showUpdateSingleMovieForm = false;
    },
  },
  mounted() {
    this.fetchMovies(); // Lấy danh sách phim
    this.fetchActorOptions();
    this.fetchcategorieOptions();
    this.fetchdirectorOptions();
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
