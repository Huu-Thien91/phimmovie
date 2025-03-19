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
  <div class="system-security-management">
    <h1>Quản lý Hệ thống & Bảo mật</h1>

    <div class="tabs">
      <button @click="selectTab('adminRoles')" :class="{ active: currentTab === 'adminRoles' }">Quản lý Quyền Hạn Admin</button>
      <button @click="selectTab('antiBot')" :class="{ active: currentTab === 'antiBot' }">Chống Bot & Bảo Vệ Nội Dung</button>
      <button @click="selectTab('loginMonitoring')" :class="{ active: currentTab === 'loginMonitoring' }">Giám Sát Đăng Nhập</button>
      <button @click="selectTab('dataBackup')" :class="{ active: currentTab === 'dataBackup' }">Sao Lưu Dữ Liệu</button>
    </div>

    <div v-if="currentTab === 'adminRoles'" class="tab-content">
      <h2>Quản lý Quyền Hạn Admin</h2>
      <table class="data-table">
        <thead>
          <tr>
            <th>Tên Admin</th>
            <th>Email</th>
            <th>Quyền Hiện Tại</th>
            <th>Thao Tác</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="admin in admins" :key="admin.id">
            <td>{{ admin.name }}</td>
            <td>{{ admin.email }}</td>
            <td>{{ admin.role }}</td>
            <td>
              <select v-model="admin.role" class="role-select">
                <option value="superAdmin">Super Admin</option>
                <option value="contentManager">Content Manager</option>
                <option value="moderator">Moderator</option>
              </select>
              <button @click="saveAdminRole(admin)" class="save-button">Lưu</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="currentTab === 'antiBot'" class="tab-content">
      <h2>Chống Bot & Bảo Vệ Nội Dung</h2>
      <p>Bảo vệ nội dung khỏi sao chép và tải xuống trái phép.</p>
      <button @click="activateAntiBot" class="action-button">Kích Hoạt Chống Bot</button>
    </div>

    <div v-if="currentTab === 'loginMonitoring'" class="tab-content">
      <h2>Giám Sát Đăng Nhập</h2>
      <table class="data-table">
        <thead>
          <tr>
            <th>Tên Admin</th>
            <th>Thời Gian Đăng Nhập</th>
            <th>Địa Chỉ IP</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="login in loginHistories" :key="login.id">
            <td>{{ login.name }}</td>
            <td>{{ login.timestamp }}</td>
            <td>{{ login.ip }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="currentTab === 'dataBackup'" class="tab-content">
      <h2>Sao Lưu Dữ Liệu</h2>
      <p>Nhấn vào nút bên dưới để sao lưu dữ liệu ngay lập tức.</p>
      <button @click="backupData" class="action-button">Sao Lưu Dữ Liệu</button>
    </div>
  </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();
const currentTab = ref('adminRoles');

const admins = ref([
  { id: 1, name: 'Nguyễn Văn A', email: 'admin1@example.com', role: 'superAdmin' },
  { id: 2, name: 'Trần Thị B', email: 'admin2@example.com', role: 'contentManager' },
]);

const loginHistories = ref([
  { id: 1, name: 'Nguyễn Văn A', timestamp: '2023-03-01 10:00', ip: '192.168.1.1' },
  { id: 2, name: 'Trần Thị B', timestamp: '2023-03-02 15:30', ip: '192.168.1.2' },
]);

const selectTab = (tab) => {
  currentTab.value = tab;
};


const saveAdminRole = (admin) => {
  alert(`Quyền của "${admin.name}" đã được cập nhật thành: ${admin.role}`);
};

const activateAntiBot = () => {
  alert('Hệ thống chống bot đã được kích hoạt thành công!');
};

const backupData = () => {
  alert('Dữ liệu đã được sao lưu thành công!');
};
</script>

<style scoped>
@import "/src/assets/css/admin.css";
body {
  font-family: 'Arial', sans-serif;
  margin: 0;
  background-color: #f5f7fa;
}

.system-security-management {
  background-color: #fff;
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  animation: fadeIn 1s ease-in-out
}

h1 {
  text-align: center;
  font-size: 28px;
  margin-bottom: 20px;
  color: #333;
}

.tabs {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-bottom: 20px;
}

.tabs button {
  padding: 10px 15px;
  background-color: #3498db;
  color: white;
  border: none;
  border-radius: 6px;
  transition: all 0.3s ease;
}

.tabs button:hover {
  background-color: #2980b9;
}

.tabs button.active {
  background-color: #2ecc71;  
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
@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.tab-content {
  padding: 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.data-table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 10px;
  border: 1px solid #ddd;
  text-align: left;
}

th {
  background: #f8f9fa;
  font-weight: bold;
}

.role-select, .action-button {
  margin-top: 10px;
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.action-button {
  background-color: #e67e22;
  color: white;
}

.action-button:hover {
  background-color: #d35400;
}

.save-button {
  background-color: #4caf50;
  color: white;
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.save-button:hover {
  background-color: #45a049;
}

.back-button {
  margin: 20px;
  background-color: #3498DB;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.back-button:hover {
  background-color: #2980B9;
}
</style>
