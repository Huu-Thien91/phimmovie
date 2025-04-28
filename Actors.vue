<template>
    <div class="container">
        <aside class="sidebar">
            <h2> Bảng điều khiển </h2>
            <ul>
                <li><router-link to="/admin/movies">Quản lý phim</router-link></li>
                <li><router-link to="/admin/actors">Quản lí diễn viên </router-link></li>
                <li><router-link to="/admin/directors">Quản lí đạo diễn </router-link></li>
                <li><router-link to="/admin/categories">Thể Loại</router-link></li>
                <li><router-link to="/">Đăng xuất</router-link></li>
            </ul>
        </aside>
        <h2 class="title">Quản lý diễn viên</h2>

        <!-- Tìm kiếm -->
        <input v-model="searchTerm" @input="searchActors" type="text" placeholder="Tìm kiếm diễn viên..."
            class="input mb-4" />

        <!-- Form Thêm / Cập nhật -->
        <div v-if="showForm" class="form">
            <h3>{{ editingActor ? 'Cập nhật' : 'Thêm' }} diễn viên</h3>
            <div class="form-grid">
                <input v-model="actorForm.nameAct" type="text" placeholder="Tên diễn viên" class="input" />
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
                    <th>Hành động</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="actor in paginatedActors" :key="actor.actorId">
                    <td>{{ actor.actorId }}</td>
                    <td>{{ actor.nameAct }}</td>
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
            <span>Trang {{ page }} / {{ totalPages }}</span>
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
            },
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
            const res = await axios.get('http://localhost:5289/api/AdminActor?sortBy=ActorId&sortDirection=asc&page=1&pageSize=100');
            this.actors = res.data;
        },
        async addActor() {
            const formData = new FormData();
            formData.append('NameAct', this.actorForm.nameAct);
            await axios.post('http://localhost:5289/api/AdminActor/AddActor', formData);
            this.fetchActors();
            this.resetForm();
            alert('Thêm diễn viên thành công!');
        },
        async deleteActor(id) {
            if (confirm('Bạn có chắc chắn muốn xóa?')) {
                await axios.delete(`http://localhost:5289/api/AdminActor/DeleteActor/${id}`);
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
            await axios.put(`http://localhost:5289/api/AdminActor/UpdateActor/${this.actorForm.actorId}`, formData);
            this.fetchActors();
            this.resetForm();
            alert('Cập nhật thành công!');
        },
        resetForm() {
            this.actorForm = {
                actorId: null,
                nameAct: '',
            };
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
@import "/src/assets/css/admin.css";

.container {
    background-color: #fff;
    border-radius: 20px;
    padding: 30px;
    box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
    animation: fadeIn 1s ease-in-out;
}

.title {
    font-size: 24px;
    font-weight: bold;
    margin-bottom: 16px;
    text-align: center;
}

.input {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    width: 100%;
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
