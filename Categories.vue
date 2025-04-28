<template>
    <div class="container">
        <aside class="sidebar">
            <h2> Bảng điều khiển </h2>
            <ul>
                <li><router-link to="/admin/movies">Quản lý phim</router-link></li>
                <li><router-link to="/admin/actors">Quản lí diễn viên </router-link></li>
                <li><router-link to="/admin/directors">Quản lí đạo diễn </router-link></li>
                <li><router-link to="/admin/categories">Thể Loại</router-link></li>
                <li><router-link to="/auth/Login">Đăng xuất</router-link></li>
            </ul>
        </aside>
        <h2 class="title">Thể loại</h2>

        <!-- Tìm kiếm -->
        <input v-model="searchTerm" @input="searchCategories" type="text" placeholder="Tìm kiếm thể loại..."
            class="input mb-16" />

        <!-- Form Thêm / Sửa thể loại -->
        <div v-if="showForm" class="form">
            <h3 class="form-title">{{ editingCategory ? "Cập nhật" : "Thêm" }} Thể loại</h3>
            <div class="form-grid">
                <input v-model="categoryForm.categoryName" type="text" placeholder="Tên thể loại" class="input" />
            </div>
            <div class="actions">
                <button @click="editingCategory ? updateCategory() : addCategory()" class="btn btn-primary">
                    {{ editingCategory ? "Cập nhật" : "Thêm" }}
                </button>
                <button @click="resetForm" class="btn btn-secondary">Hủy</button>
            </div>
        </div>

        <button @click="toggleForm" class="btn btn-green mb-16">
            {{ showForm ? "Ẩn form" : "Thêm thể loại" }}
        </button>

        <!-- Danh sách thể loại -->
        <table class="table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Tên thể loại</th>
                    <th>Hành động</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="cat in paginatedCategories" :key="cat.categoryId">
                    <td>{{ cat.categoryId }}</td>
                    <td>{{ cat.categoryName }}</td>
                    <td>
                        <button @click="editCategory(cat)" class="btn btn-secondary">Sửa</button>
                        <button @click="deleteCategory(cat.categoryId)" class="btn btn-danger">Xóa</button>
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
            categories: [],
            categoryForm: {
                categoryId: null,
                categoryName: "",
            },
            showForm: false,
            editingCategory: false,
            searchTerm: "",
            page: 1,
            perPage: 10,
        };
    },
    computed: {
        filteredCategories() {
            if (!this.searchTerm) return this.categories;
            return this.categories.filter((cat) =>
                cat.categoryName.toLowerCase().includes(this.searchTerm.toLowerCase())
            );
        },
        paginatedCategories() {
            const start = (this.page - 1) * this.perPage;
            return this.filteredCategories.slice(start, start + this.perPage);
        },
        totalPages() {
            return Math.ceil(this.filteredCategories.length / this.perPage);
        },
    },
    created() {
        this.fetchCategories();
    },
    methods: {
        async fetchCategories() {
            try {
                const res = await axios.get(
                    "http://localhost:5289/api/AdminCategories?sortBy=CategoryId&sortDirection=asc&page=1&pageSize=100"
                );
                this.categories = res.data;
            } catch (error) {
                console.error("Lỗi tải danh sách thể loại:", error);
                alert("Không thể tải danh sách thể loại");
            }
        },
        async addCategory() {
            if (!this.categoryForm.categoryName) {
                alert("Vui lòng nhập tên thể loại!");
                return;
            }
            try {
                // API thêm thể loại truyền categoryName qua query string
                await axios.post(
                    `http://localhost:5289/api/AdminCategories?categoryName=${encodeURIComponent(this.categoryForm.categoryName)}`
                );
                await this.fetchCategories();
                this.resetForm();
                alert("Thêm thể loại thành công!");
            } catch (error) {
                console.error("Lỗi khi thêm thể loại:", error);
                alert("Thêm thể loại thất bại!");
            }
        },
        async updateCategory() {
            if (!this.categoryForm.categoryName) {
                alert("Vui lòng nhập tên thể loại!");
                return;
            }
            try {
                // API sửa: PUT với query string categoryName
                await axios.put(
                    `http://localhost:5289/api/AdminCategories/${this.categoryForm.categoryId}?categoryName=${encodeURIComponent(this.categoryForm.categoryName)}`
                );
                await this.fetchCategories();
                this.resetForm();
                alert("Cập nhật thể loại thành công!");
            } catch (error) {
                console.error("Lỗi khi cập nhật thể loại:", error);
                alert("Cập nhật thể loại thất bại!");
            }
        },
        async deleteCategory(id) {
            if (!confirm("Bạn có chắc chắn muốn xóa thể loại này?")) return;
            try {
                const res = await axios.delete(
                    `http://localhost:5289/api/AdminCategories/${id}`,
                    { headers: { Accept: "application/json" } } // thêm header nếu cần
                );
                console.log("Delete response:", res.data);
                await this.fetchCategories();
                alert("Xóa thể loại thành công!");
            } catch (error) {
                console.error("Lỗi khi xóa thể loại:", error.response || error.message);
                alert("Xóa thể loại thất bại!");
            }
        },

        editCategory(cat) {
            this.categoryForm = { ...cat };
            this.editingCategory = true;
            this.showForm = true;
        },
        resetForm() {
            this.categoryForm = {
                categoryId: null,
                categoryName: "",
            };
            this.editingCategory = false;
            this.showForm = false;
        },
        toggleForm() {
            this.showForm = !this.showForm;
            if (!this.showForm) this.resetForm();
        },
        searchCategories() {
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
    width: 2000%;
    margin-right: 200px;
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
    font-size: 14px;
}

.input.mb-16 {
    margin-bottom: 16px;
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
    grid-template-columns: 1fr;
    gap: 10px;
    margin-bottom: 12px;
}

@media (min-width: 768px) {
    .form-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}

.actions {
    margin-top: 10px;
}

.btn {
    padding: 8px 12px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin-right: 8px;
    font-size: 14px;
}

.btn.btn-primary {
    background-color: #3490dc;
    color: #fff;
}

.btn.btn-secondary {
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

.avatar {
    width: 40px;
    height: 40px;
    border-radius: 50%;
    object-fit: cover;
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
