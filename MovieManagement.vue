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
        <h2 class="title">Quản lý Đạo diễn</h2>

        <!-- Phần tìm kiếm -->
        <input v-model="searchTerm" @input="searchDirectors" type="text" placeholder="Tìm kiếm đạo diễn..."
            class="input mb-4" />

        <!-- Form Thêm / Cập nhật -->
        <div v-if="showForm" class="form">
            <h3 class="form-title">{{ editingDirector ? 'Cập nhật' : 'Thêm' }} Đạo diễn</h3>
            <div class="form-grid">
                <input v-model="directorForm.nameDir" type="text" placeholder="Tên đạo diễn" class="input" />
            </div>
            <div class="form-actions">
                <button @click="editingDirector ? updateDirector() : addDirector()" class="btn btn-blue">
                    {{ editingDirector ? 'Cập nhật' : 'Thêm' }}
                </button>
                <button @click="resetForm()" class="btn btn-gray">Hủy</button>
            </div>
        </div>

        <button @click="toggleForm" class="btn btn-green mb-4">
            {{ showForm ? 'Ẩn form' : 'Thêm đạo diễn' }}
        </button>

        <!-- Bảng Danh sách Đạo diễn -->
        <table class="table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Tên</th>
                    <th>Hành động</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="director in paginatedDirectors" :key="director.directorID">
                    <td>{{ director.directorID }}</td>
                    <td>{{ director.nameDir }}</td>
                    <td>
                        <button @click="editDirector(director)" class="btn btn-gray">Sửa</button>
                        <button @click="deleteDirector(director.directorID)" class="btn btn-red">Xóa</button>
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
import axios from "axios";

export default {
    data() {
        return {
            directors: [],
            directorForm: {
                directorID: null,
                nameDir: "",
            },
            editingDirector: false,
            showForm: false,
            searchTerm: "",
            page: 1,
            perPage: 10,
        };
    },
    computed: {
        filteredDirectors() {
            if (!this.searchTerm) return this.directors;
            return this.directors.filter(director =>
                director.nameDir.toLowerCase().includes(this.searchTerm.toLowerCase())
            );
        },
        paginatedDirectors() {
            const start = (this.page - 1) * this.perPage;
            return this.filteredDirectors.slice(start, start + this.perPage);
        },
        totalPages() {
            return Math.ceil(this.filteredDirectors.length / this.perPage);
        },
    },
    created() {
        this.fetchDirectors();
    },
    methods: {
        async fetchDirectors() {
            try {
                const res = await axios.get(
                    "http://localhost:5289/api/AdminDirectors/List-Directors?sortBy=DirectorId&sortDirection=asc&page=1&pageSize=100"
                );
                this.directors = res.data;
            } catch (error) {
                console.error("Lỗi tải danh sách đạo diễn:", error);
                alert("Không thể tải danh sách đạo diễn");
            }
        },
        async addDirector() {
            try {
                const formData = new FormData();
                formData.append("NameDir", this.directorForm.nameDir);
                await axios.post("http://localhost:5289/api/AdminDirectors/AddDirector", formData);
                await this.fetchDirectors();
                this.resetForm();
                alert("Thêm đạo diễn thành công!");
            } catch (error) {
                console.error("Lỗi khi thêm đạo diễn:", error);
                alert("Thêm đạo diễn thất bại!");
            }
        },
        async updateDirector() {
            try {
                const formData = new FormData();
                formData.append("NameDir", this.directorForm.nameDir);
                await axios.put(
                    `http://localhost:5289/api/AdminDirectors/UpdateDirector/${this.directorForm.directorID}`,
                    formData
                );
                await this.fetchDirectors();
                this.resetForm();
                alert("Cập nhật đạo diễn thành công!");
            } catch (error) {
                console.error("Lỗi khi cập nhật đạo diễn:", error);
                alert("Cập nhật đạo diễn thất bại!");
            }
        },
        async deleteDirector(id) {
            if (confirm("Bạn có chắc chắn muốn xóa?")) {
                try {
                    await axios.delete(`http://localhost:5289/api/AdminDirectors/DeleteDirector/${id}`);
                    await this.fetchDirectors();
                    alert("Xóa đạo diễn thành công!");
                } catch (error) {
                    console.error("Lỗi khi xóa đạo diễn:", error);
                    alert("Xóa đạo diễn thất bại!");
                }
            }
        },
        editDirector(director) {
            this.directorForm = { ...director };
            this.editingDirector = true;
            this.showForm = true;
        },
        resetForm() {
            this.directorForm = {
                directorID: null,
                nameDir: "",
            };
            this.editingDirector = false;
            this.showForm = false;
        },
        toggleForm() {
            this.showForm = !this.showForm;
            if (!this.showForm) this.resetForm();
        },
        searchDirectors() {
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

.form {
    background-color: #f9f9f9;
    padding: 16px;
    border-radius: 6px;
    margin-bottom: 16px;
    border: 1px solid #ddd;
}

.form-title {
    font-size: 18px;
    font-weight: 600;
    margin-bottom: 12px;
}

.form-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
    margin-bottom: 12px;
}

.form-actions {
    margin-top: 10px;
}

.btn {
    padding: 8px 12px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin-right: 8px;
}

.btn.btn-blue {
    background-color: #3490dc;
    color: #fff;
}

.btn.btn-gray {
    background-color: #ccc;
    color: #333;
}

.btn.btn-red {
    background-color: #e3342f;
    color: #fff;
}

.btn.btn-green {
    background-color: #38c172;
    color: #fff;
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
    align-items: center;
    margin-top: 16px;
    gap: 10px;
}

.pagination button {
    padding: 6px 12px;
    border: 1px solid #ccc;
    border-radius: 4px;
    background-color: #fff;
    cursor: pointer;
}

.pagination button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}
</style>
