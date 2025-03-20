<template>
  <div class="container">
    <aside class="sidebar">
      <h2>Bảng điều khiển</h2>
      <ul>
        <li><router-link to="/admin/movies"><i class="fa fa-film"></i> Quản lý phim</router-link></li>
        <li><router-link to="/admin/finance"><i class="fa fa-money"></i> Quản lí tài chính</router-link></li>
        <li><router-link to="/admin/vipmanagenment"><i class="fa fa-user"></i> Quản lí tài khoản VIP và Thanh
            Toán</router-link></li>
        <li><router-link to="/admin/account"><i class="fa fa-lock"></i> Quản lí hệ thống và bảo mật
            (admin)</router-link></li>
        <li><router-link to="/admin/user"><i class="fa fa-users"></i> Quản lí người dùng</router-link></li>
        <li><router-link to="/admin/transactions"><i class="fa fa-history"></i> Lịch sử giao dịch</router-link></li>
        <li><router-link to="/admin/setting"><i class="fa fa-cog"></i> Cài đặt chung</router-link></li>
        <li><router-link to="/login"><i class="fa fa-sign-out"></i> Đăng xuất</router-link></li>
      </ul>
    </aside>

    <!-- Quản lý người dùng -->
    <div class="user-management">
      <h1>Quản lý Người Dùng</h1>

      <!-- Toggle chế độ sáng/tối -->
      <button @click="toggleTheme" class="toggle-mode">
        Chuyển đổi chế độ
      </button>

      <!-- Form thêm người dùng -->
      <div class="add-user">
        <h2>Thêm người dùng</h2>
        <input v-model="newUser.username" placeholder="Tên đăng nhập" />
        <input v-model="newUser.email" placeholder="Email" />
        <button @click="addUser" class="add-button">Thêm</button>
      </div>

      <!-- Danh sách người dùng -->
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
              <td>
                <template v-if="editingUserId === user.id">
                  <input v-model="editUser.username" />
                </template>
                <template v-else>{{ user.username }}</template>
              </td>
              <td>
                <template v-if="editingUserId === user.id">
                  <input v-model="editUser.email" />
                </template>
                <template v-else>{{ user.email }}</template>
              </td>
              <td>{{ user.status }}</td>
              <td>
                <template v-if="editingUserId === user.id">
                  <button @click="saveEdit(user.id)" class="save-button">Lưu</button>
                  <button @click="cancelEdit" class="cancel-button">Hủy</button>
                </template>
                <template v-else>
                  <button @click="startEdit(user)" class="edit-button">Sửa</button>
                  <button @click="deleteUser(user.id)" class="delete-button">Xóa</button>
                </template>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

// Danh sách người dùng
const users = ref([
  { id: "U001", username: "NguyenVanA", email: "nguyenvana@example.com", status: "Hoạt động" },
  { id: "U002", username: "TranThiB", email: "tranthib@example.com", status: "Đã khóa" },
]);

// Dữ liệu người dùng mới
const newUser = ref({ username: "", email: "" });

// Người dùng chỉnh sửa
const editingUserId = ref(null);
const editUser = ref({ username: "", email: "" });

// Chế độ sáng/tối
const isDarkMode = ref(false);
const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value;
  document.body.classList.toggle('dark-mode', isDarkMode.value);
};

// Thêm người dùng
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

// Bắt đầu chỉnh sửa
const startEdit = (user) => {
  editingUserId.value = user.id;
  editUser.value = { ...user };
};

// Lưu chỉnh sửa
const saveEdit = (id) => {
  const index = users.value.findIndex((user) => user.id === id);
  if (index !== -1) {
    users.value[index].username = editUser.value.username;
    users.value[index].email = editUser.value.email;
    editingUserId.value = null;
    alert("Cập nhật thông tin thành công!");
  }
};

// Hủy chỉnh sửa
const cancelEdit = () => {
  editingUserId.value = null;
};

// Xóa người dùng
const deleteUser = (id) => {
  const confirmed = confirm("Bạn có chắc chắn muốn xóa người dùng này?");
  if (confirmed) {
    users.value = users.value.filter((user) => user.id !== id);
    alert("Người dùng đã bị xóa!");
  }
};
</script>

<style scoped>
@import "/src/assets/css/admin.css";

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
  transition: transform 0.3s ease;
}

.add-button:hover {
  transform: scale(1.05);
}

/* Giao diện bảng danh sách */
.user-list table {
  width: 100%;
  border-collapse: collapse;
}

.user-list th,
.user-list td {
  padding: 10px;
  border: 1px solid #ddd;
}

.user-list th {
  background-color: #f4f4f4;
}

.edit-button,
.delete-button,
.save-button,
.cancel-button {
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  color: white;
  font-size: 14px;
  transition: transform 0.3s ease;
}

.edit-button:hover,
.delete-button:hover,
.save-button:hover,
.cancel-button:hover {
  transform: scale(1.05);
}

.edit-button {
  background-color: #007bff;
}

.delete-button {
  background-color: #dc3545;
}

.save-button {
  background-color: #28a745;
}

.cancel-button {
  background-color: #6c757d;
}

.user-management {
  background: linear-gradient(to right, #f9f9f9, #e6e6e6);
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  animation: fadeIn 1s ease-in-out;
  width: 120%;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
  color: #34495e;
}

body.dark-mode {
  background-color: #121212;
  color: #ffffff;
}

.toggle-mode {
  margin-bottom: 20px;
  padding: 8px 16px;
  background-color: #e67e22;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.toggle-mode:hover {
  background-color: #d35400;
}
</style>
