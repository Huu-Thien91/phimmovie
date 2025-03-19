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
    <div class="transaction-history">
      <h1>Lịch Sử Giao Dịch</h1>

      <!-- Bộ lọc -->
      <div class="filters">
        <div class="form-group">
          <label for="filterUser">Người dùng</label>
          <input type="text" id="filterUser" v-model="filters.user" placeholder="Tìm kiếm theo tên người dùng" />
        </div>
        <div class="form-group">
          <label for="filterMethod">Phương thức thanh toán</label>
          <select id="filterMethod" v-model="filters.paymentMethod">
            <option value="">Tất cả</option>
            <option value="Momo">Momo</option>
            <option value="VNPay">VNPay</option>
            <option value="ZaloPay">ZaloPay</option>
            <option value="Credit Card">Thẻ tín dụng</option>
          </select>
        </div>
        <button @click="applyFilters" class="filter-button">Lọc</button>
      </div>

      <!-- Bảng lịch sử giao dịch -->
      <table class="transaction-table">
        <thead>
          <tr>
            <th>ID Giao Dịch</th>
            <th>Người Dùng</th>
            <th>Số Tiền</th>
            <th>Phương Thức</th>
            <th>Ngày Giao Dịch</th>
            <th>Trạng Thái</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="transaction in filteredTransactions" :key="transaction.id">
            <td>{{ transaction.id }}</td>
            <td>{{ transaction.user }}</td>
            <td>{{ transaction.amount }} VND</td>
            <td>{{ transaction.paymentMethod }}</td>
            <td>{{ transaction.date }}</td>
            <td
              :class="{ 'success': transaction.status === 'Thành công', 'failed': transaction.status === 'Thất bại' }">
              {{ transaction.status }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

// Dữ liệu giao dịch mẫu
const transactions = ref([
  { id: 'T001', user: 'Nguyễn Văn A', amount: 500000, paymentMethod: 'Momo', date: '2023-04-01', status: 'Thành công' },
  { id: 'T002', user: 'Trần Thị B', amount: 300000, paymentMethod: 'ZaloPay', date: '2023-04-03', status: 'Thành công' },
  { id: 'T003', user: 'Phạm Văn C', amount: 700000, paymentMethod: 'VNPay', date: '2023-04-05', status: 'Thất bại' },
  { id: 'T004', user: 'Lê Thị D', amount: 1000000, paymentMethod: 'Credit Card', date: '2023-04-06', status: 'Thất bại' },
  { id: 'T005', user: 'Nguyễn Hữu Thiện', amount: 2000000, paymentMethod: 'Momo', date: '2023-05-06', status: 'Thành công' },
  { id: 'T006', user: 'Nguyễn Hữu Đăng', amount: 3000000, paymentMethod: 'ZaloPay', date: '2023-02-28', status: 'Thành công' },
  { id: 'T007', user: 'Nguyễn Hữu Khoa', amount: 4000000, paymentMethod: 'ZaloPay', date: '2023-02-28', status: 'Thành công' },
]);

// Bộ lọc
const filters = ref({
  user: '',
  paymentMethod: '',
});

const filteredTransactions = ref(transactions.value);

// Áp dụng bộ lọc
const applyFilters = () => {
  filteredTransactions.value = transactions.value.filter((transaction) => {
    const matchesUser = filters.value.user
      ? transaction.user.toLowerCase().includes(filters.value.user.toLowerCase())
      : true;
    const matchesPaymentMethod = filters.value.paymentMethod
      ? transaction.paymentMethod === filters.value.paymentMethod
      : true;
    return matchesUser && matchesPaymentMethod;
  });
};
</script>

<style scoped>
@import "/src/assets/css/admin.css";
body {
  font-family: 'Arial', sans-serif;
  background-color: #f8f9fa;
  margin: 0;
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

.transaction-history {
  background-color: #fff;
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  animation: fadeIn 1s ease-in-out
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

h1 {
  text-align: center;
  margin-bottom: 20px;
  color: #2c3e50;
}

.filters {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
}

.form-group {
  flex: 1;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input,
select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
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

.transaction-table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  border: 1px solid #ddd;
  padding: 10px;
  text-align: left;
}

th {
  background-color: #f4f4f4;
}

td.success {
  color: #27ae60;
  font-weight: bold;
}

td.failed {
  color: #e74c3c;
  font-weight: bold;
}
</style>
