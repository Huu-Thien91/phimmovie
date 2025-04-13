<template>
    <div class="container">
      <h2 class="title">Quản lý diễn viên</h2>
  
      <!-- Tìm kiếm -->
      <input
        v-model="searchTerm"
        @input="searchActors"
        type="text"
        placeholder="Tìm kiếm diễn viên..."
        class="input mb-4"
      />
  
      <!-- Form Thêm / Cập nhật -->
      <div v-if="showForm" class="form">
        <h3>{{ editingActor ? 'Cập nhật' : 'Thêm' }} diễn viên</h3>
        <div class="form-grid">
          <input v-model="actorForm.nameAct" type="text" placeholder="Tên diễn viên" class="input" />
          <input v-model="actorForm.nationality" type="text" placeholder="Quốc tịch" class="input" />
          <input v-model="actorForm.professional" type="text" placeholder="Chuyên môn" class="input" />
          <input type="file" @change="handleFileUpload" class="input" />
          <textarea v-model="actorForm.description" placeholder="Mô tả" class="input full"></textarea>
        </div>
        <div class="actions">
          <button @click="editingActor ? updateActor() : addActor()" class="btn blue">
            {{ editingActor ? 'Cập nhật' : 'Thêm' }}
          </button>
          <button @click="resetForm()" class="btn gray">Hủy</button>
        </div>
      </div>
  
      <button @click="toggleForm" class="btn green mb-4">
        {{ showForm ? 'Ẩn form' : 'Thêm diễn viên' }}
      </button>
  
      <!-- Danh sách diễn viên -->
      <table class="table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Tên</th>
            <th>Quốc tịch</th>
            <th>Chuyên môn</th>
            <th>Avatar</th>
            <th>Hành động</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="actor in paginatedActors" :key="actor.actorId">
            <td>{{ actor.actorId }}</td>
            <td>{{ actor.nameAct }}</td>
            <td>{{ actor.nationality }}</td>
            <td>{{ actor.professional }}</td>
            <td><img :src="actor.avatarUrl" class="avatar" /></td>
            <td>
              <button @click="editActor(actor)" class="btn gray">Sửa</button>
              <button @click="deleteActor(actor.actorId)" class="btn red">Xóa</button>
            </td>
          </tr>
        </tbody>
      </table>
  
      <!-- Phân trang -->
      <div class="pagination">
        <button @click="prevPage" :disabled="page === 1">&laquo;</button>
        <span>Trang {{ page }}</span>
        <button @click="nextPage" :disabled="page >= totalPages">&raquo;</button>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        actors: [],
        actorForm: {
          actorId: null,
          nameAct: '',
          description: '',
          nationality: '',
          professional: '',
        },
        avatarFile: null,
        editingActor: false,
        showForm: false,
        searchTerm: '',
        page: 1,
        perPage: 10,
      };
    },
    computed: {
      filteredActors() {
        if (!this.searchTerm) return this.actors;
        return this.actors.filter(actor =>
          actor.nameAct.toLowerCase().includes(this.searchTerm.toLowerCase())
        );
      },
      paginatedActors() {
        const start = (this.page - 1) * this.perPage;
        return this.filteredActors.slice(start, start + this.perPage);
      },
      totalPages() {
        return Math.ceil(this.filteredActors.length / this.perPage);
      },
    },
    created() {
      this.fetchActors();
    },
    methods: {
      async fetchActors() {
        const res = await axios.get('http://localhost:5148/api/AdminActor?sortBy=ActorId&sortDirection=asc&page=1&pageSize=100');
        this.actors = res.data;
      },
      handleFileUpload(event) {
        this.avatarFile = event.target.files[0];
      },
      async addActor() {
        const formData = new FormData();
        formData.append('NameAct', this.actorForm.nameAct);
        formData.append('Description', this.actorForm.description);
        formData.append('Nationality', this.actorForm.nationality);
        formData.append('Professional', this.actorForm.professional);
        if (this.avatarFile) {
          formData.append('AvatarUrlFile', this.avatarFile);
        }
        await axios.post('http://localhost:5148/api/AdminActor/AddActor', formData);
        this.fetchActors();
        this.resetForm();
        alert('Thêm diễn viên thành công!');
      },
      async deleteActor(id) {
        if (confirm('Bạn có chắc chắn muốn xóa?')) {
          await axios.delete(`http://localhost:5148/api/AdminActor/DeleteActor/${id}`);
          this.fetchActors();
          alert('Xóa thành công!');
        }
      },
      editActor(actor) {
        this.actorForm = { ...actor };
        this.editingActor = true;
        this.showForm = true;
      },
      async updateActor() {
        const formData = new FormData();
        formData.append('NameAct', this.actorForm.nameAct);
        formData.append('Description', this.actorForm.description);
        formData.append('Nationality', this.actorForm.nationality);
        formData.append('Professional', this.actorForm.professional);
        if (this.avatarFile) {
          formData.append('AvatarUrlFile', this.avatarFile);
        }
        await axios.put(`http://localhost:5148/api/AdminActor/UpdateActor/${this.actorForm.actorId}`, formData);
        this.fetchActors();
        this.resetForm();
        alert('Cập nhật thành công!');
      },
      resetForm() {
        this.actorForm = {
          actorId: null,
          nameAct: '',
          description: '',
          nationality: '',
          professional: '',
        };
        this.avatarFile = null;
        this.editingActor = false;
        this.showForm = false;
      },
      toggleForm() {
        this.showForm = !this.showForm;
        if (!this.showForm) this.resetForm();
      },
      searchActors() {
        this.page = 1;
      },
      nextPage() {
        if (this.page < this.totalPages) this.page++;
      },
      prevPage() {
        if (this.page > 1) this.page--;
      },
    },
  };
  </script>
  
  <style scoped>
  .container {
    max-width: 1000px;
    margin: 20px auto;
    padding: 20px;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  }
  .title {
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 16px;
  }
  .input {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    width: 100%;
  }
  .input.full {
    grid-column: span 2;
  }
  .form-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
  }
  .actions {
    margin-top: 10px;
  }
  .btn {
    padding: 8px 16px;
    border-radius: 4px;
    border: none;
    cursor: pointer;
    margin-right: 8px;
  }
  .btn.blue {
    background-color: #3490dc;
    color: white;
  }
  .btn.gray {
    background-color: #ccc;
  }
  .btn.red {
    background-color: #e3342f;
    color: white;
  }
  .btn.green {
    background-color: #38c172;
    color: white;
  }
  .table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 10px;
  }
  .table th,
  .table td {
    padding: 8px;
    border: 1px solid #ddd;
    text-align: center;
  }
  .avatar {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    object-fit: cover;
  }
  .pagination {
    display: flex;
    justify-content: center;
    margin-top: 16px;
    gap: 10px;
    align-items: center;
  }
  .pagination button {
    padding: 6px 12px;
    border: 1px solid #ccc;
    border-radius: 4px;
    background-color: white;
    cursor: pointer;
  }
  .pagination button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
  </style>
  