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
    <div class="user-management">
      <h1>Quản lý Người Dùng</h1>

      <!-- Danh sách người dùng -->
      <div class="user-list">
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
              <td :class="{ 'active': user.status === 'Hoạt động', 'blocked': user.status === 'Đã khóa' }">
                {{ user.status }}
              </td>
              <td>
                <button @click="viewUserDetails(user)" class="details-button">Xem Chi Tiết</button>
                <button @click="toggleUserStatus(user)" class="toggle-button">
                  {{ user.status === 'Hoạt động' ? 'Khóa' : 'Mở khóa' }}
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Chi tiết người dùng -->
      <div v-if="selectedUser" class="user-details">
        <h2>Chi Tiết Người Dùng</h2>
        <p><strong>User ID:</strong> {{ selectedUser.id }}</p>
        <p><strong>Username:</strong> {{ selectedUser.username }}</p>
        <p><strong>Email:</strong> {{ selectedUser.email }}</p>
        <p><strong>Trạng Thái:</strong> {{ selectedUser.status }}</p>

        <!-- Lịch sử thanh toán -->
        <h3>Lịch Sử Thanh Toán</h3>
        <table>
          <thead>
            <tr>
              <th>ID Giao Dịch</th>
              <th>Số Tiền</th>
              <th>Ngày Giao Dịch</th>
              <th>Phương Thức</th>
              <th>Trạng Thái</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="transaction in selectedUser.transactions" :key="transaction.id">
              <td>{{ transaction.id }}</td>
              <td>{{ transaction.amount }} VND</td>
              <td>{{ transaction.date }}</td>
              <td>{{ transaction.method }}</td>
              <td
                :class="{ 'success': transaction.status === 'Thành công', 'failed': transaction.status === 'Thất bại' }">
                {{ transaction.status }}
              </td>
            </tr>
          </tbody>
        </table>
        <button @click="closeUserDetails" class="close-button">Đóng</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

// Danh sách người dùng mẫu
const users = ref([
  {
    id: 'U001',
    username: 'NguyenVanA',
    email: 'nguyenvana@example.com',
    status: 'Hoạt động',
    transactions: [
      { id: 'T001', amount: 500000, date: '2023-04-01', method: 'Momo', status: 'Thành công' },
      { id: 'T002', amount: 300000, date: '2023-04-05', method: 'VNPay', status: 'Thành công' },
    ],
  },
  {
    id: 'U002',
    username: 'TranThiB',
    email: 'tranthib@example.com',
    status: 'Đã khóa',
    transactions: [
      { id: 'T003', amount: 700000, date: '2023-03-15', method: 'Creat Card', status: 'Thành công' },
    ],
  },
]);

// Người dùng được chọn
const selectedUser = ref(null);

// Xem chi tiết người dùng
const viewUserDetails = (user) => {
  selectedUser.value = user;
};

// Đóng chi tiết người dùng
const closeUserDetails = () => {
  selectedUser.value = null;
};

// Khóa / mở khóa tài khoản người dùng
const toggleUserStatus = (user) => {
  user.status = user.status === 'Hoạt động' ? 'Đã khóa' : 'Hoạt động';
  alert(`Trạng thái của người dùng "${user.username}" đã thay đổi thành "${user.status}".`);
};
</script>

<style scoped>
@import "/src/assets/css/admin.css";

.user-management {
  background-color: #fff;
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  animation: fadeIn 1s ease-in-out;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
  color: #34495e;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px 0;
}

th,
td {
  padding: 12px;
  text-align: left;
  border: 1px solid #ddd;
}

th {
  background-color: #f4f4f4;
  font-weight: bold;
}

td.active {
  color: #2ecc71;
}

td.blocked {
  color: #e74c3c;
}

.details-button {
  background-color: #3498db;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
}

.details-button:hover {
  background-color: #2980b9;
}

.toggle-button {
  background-color: #e67e22;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
}

.toggle-button:hover {
  background-color: #d35400;
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

.user-details {
  margin-top: 20px;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.user-details h2 {
  color: #34495e;
  margin-bottom: 15px;
}

.close-button {
  background-color: #e74c3c;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 20px;
}

.close-button:hover {
  background-color: #c0392b;
}
</style>
