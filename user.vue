<template>
  <div class="container">
    <!-- Sidebar -->
    <aside class="sidebar">
      <h2>Bảng điều khiển</h2>
      <ul>
        <li><router-link to="/admin/movies"><i class="fa fa-film"></i> Quản lý phim</router-link></li>
        <li><router-link to="/admin/finance"><i class="fa fa-money"></i> Quản lí tài chính</router-link></li>
        <li><router-link to="/admin/vipmanagenment"><i class="fa fa-user"></i> Quản lí tài khoản VIP và Thanh
            Toán</router-link></li>
        <li><router-link to="/admin/account"><i class="fa fa-lock"></i> Quản lí hệ thống và bảo mật</router-link></li>
        <li><router-link to="/admin/user"><i class="fa fa-users"></i> Quản lí người dùng</router-link></li>
        <li><router-link to="/admin/transactions"><i class="fa fa-history"></i> Lịch sử giao dịch</router-link></li>
        <li><router-link to="/admin/setting"><i class="fa fa-cog"></i> Cài đặt chung</router-link></li>
        <li><router-link to="/login"><i class="fa fa-sign-out"></i> Đăng xuất</router-link></li>
      </ul>
    </aside>

    <!-- User Management -->
    <div class="user-management">
      <h1>Quản lý Người Dùng</h1>

      <!-- Add New User Form -->
      <div class="add-user">
        <h2>Thêm người dùng</h2>
        <input v-model="newUser.username" placeholder="Tên đăng nhập" />
        <input v-model="newUser.email" placeholder="Email" />
        <button @click="addUser" class="add-button">Thêm</button>
      </div>

      <!-- User List -->
      <div class="user-list">
        <h2>Danh sách người dùng</h2>
        <table>
          <thead>
            <tr>
              <th>User ID</th>
              <th>Username</th>
              <th>Email</th>
              <th>Trạng Thái</th>
              <th>Hành Động</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="user in users" :key="user.id">
              <td>{{ user.id }}</td>
              <td>{{ user.username }}</td>
              <td>{{ user.email }}</td>
              <td>{{ user.status }}</td>
              <td>
                <button @click="startEdit(user)" class="edit-button">Sửa</button>
                <button @click="deleteUser(user.id)" class="delete-button">Xóa</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Edit Form -->
    <div v-if="isEditing" class="edit-form-overlay">
      <div class="edit-form">
        <h2>Chỉnh sửa thông tin người dùng</h2>
        <div class="form-group">
          <label>Tên đăng nhập:</label>
          <input v-model="editUser.username" />
        </div>
        <div class="form-group">
          <label>Email:</label>
          <input v-model="editUser.email" />
        </div>
        <div class="form-buttons">
          <button @click="saveEdit" class="save-button">Lưu</button>
          <button @click="cancelEdit" class="cancel-button">Hủy</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

// User List Data
const users = ref([
  { id: "U001", username: "NguyenVanA", email: "nguyenvana@example.com", status: "Hoạt động" },
  { id: "U002", username: "TranThiB", email: "tranthib@example.com", status: "Đã khóa" },
]);

// New User Data
const newUser = ref({ username: "", email: "" });

// Edit User Logic
const isEditing = ref(false);
const editUser = ref({});
let editingUserId = null;

// Add User
const addUser = () => {
  if (newUser.value.username && newUser.value.email) {
    const newId = `U00${users.value.length + 1}`;
    users.value.push({ id: newId, ...newUser.value, status: "Hoạt động" });
    newUser.value = { username: "", email: "" };
    alert("Người dùng mới đã được thêm!");
  } else {
    alert("Vui lòng nhập đầy đủ thông tin!");
  }
};

// Start Edit
const startEdit = (user) => {
  editingUserId = user.id;
  editUser.value = { ...user };
  isEditing.value = true;
};

// Save Edit
const saveEdit = () => {
  const index = users.value.findIndex((user) => user.id === editingUserId);
  if (index !== -1) {
    users.value[index].username = editUser.value.username;
    users.value[index].email = editUser.value.email;
    isEditing.value = false;
    editingUserId = null;
    alert("Cập nhật thông tin thành công!");
  }
};

// Cancel Edit
const cancelEdit = () => {
  isEditing.value = false;
  editingUserId = null;
};

// Delete User
const deleteUser = (id) => {
  if (confirm("Bạn có chắc chắn muốn xóa người dùng này?")) {
    users.value = users.value.filter((user) => user.id !== id);
    alert("Người dùng đã bị xóa!");
  }
};
</script>

<style scoped>
@import "/src/assets/css/admin.css";
/* Sidebar */

/* User Management */
.user-management {
  background-color: #fff;
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  animation: fadeIn 1s ease-in-out;
  width: 120%;
}

.user-list table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.user-list th,
.user-list td {
  border: 1px solid #ddd;
  padding: 8px;
}

.user-list th {
  background-color: #f2f2f2;
  text-align: left;
}

/* Add User */
.add-user {
  margin-bottom: 20px;
  background: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.add-user input {
  margin-right: 10px;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.add-button {
  background-color: #28a745;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
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

.add-button:hover {
  background-color: #218838;
}

/* Edit Form */
.edit-form-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
}

.edit-form {
  background-color: white;
  padding: 20px;
  width: 400px;
  border-radius: 5px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
}

.edit-form h2 {
  margin-top: 0;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  font-weight: bold;
  margin-bottom: 5px;
}

.form-group input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.form-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.save-button {
  background-color: #28a745;
  color: white;
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
}

.save-button:hover {
  background-color: #218838;
}

.cancel-button {
  background-color: #dc3545;
  color: white;
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
}

.cancel-button:hover {
  background-color: #c82333;
}

.delete-button {
  background-color: red;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 6px;
  cursor: pointer;
  margin-right: 5px;
}
.edit-button {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 6px;
  cursor: pointer;
  margin-right: 5px;
}
</style>
