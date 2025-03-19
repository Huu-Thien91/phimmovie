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
    <div class="finance-management">
      <h1>Quản lý Tài chính</h1>
      <div class="tabs">
        <button @click="selectTab('revenue')" :class="{ active: currentTab === 'revenue' }">Doanh thu</button>
        <button @click="selectTab('payments')" :class="{ active: currentTab === 'payments' }">Thanh toán</button>
        <button @click="selectTab('reports')" :class="{ active: currentTab === 'reports' }">Báo cáo tài chính</button>
      </div>

      <div v-if="currentTab === 'revenue'" class="tab-content">
        <h2>Doanh thu</h2>
        <div class="revenue-summary">
          <p class="total-revenue">Tổng doanh thu: {{ totalRevenue }}</p>
        </div>
        <ul class="transaction-list">
          <li v-for="transaction in transactions" :key="transaction.id">
            {{ transaction.date }} - {{ transaction.amount }} VND
          </li>
        </ul>
      </div>

      <div v-if="currentTab === 'payments'" class="tab-content">
        <h2>Thanh toán</h2>
        <ul class="payment-list">
          <li v-for="payment in payments" :key="payment.id">
            {{ payment.method }} - {{ payment.date }} - {{ payment.amount }} VND
          </li>
        </ul>
      </div>

      <div v-if="currentTab === 'reports'" class="tab-content">
        <h2>Báo cáo tài chính</h2>
        <ul class="report-list">
          <li v-for="report in reports" :key="report.id">
            <a :href="report.url" target="_blank">{{ report.date }} - Xem báo cáo</a>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();

// Tab hiện tại
const currentTab = ref('revenue');

// Dữ liệu mẫu
const totalRevenue = ref('1,000,000 VND');
const transactions = ref([
  { id: 1, date: '2023-01-01', amount: '500,000' },
  { id: 2, date: '2023-02-01', amount: '300,000' },
  { id: 3, date: '2023-03-01', amount: '200,000' },
]);

const payments = ref([
  { id: 1, method: 'Credit Card', date: '2023-01-01', amount: '500,000' },
  { id: 2, method: 'Paypal', date: '2023-02-01', amount: '300,000' },
  { id: 3, method: 'Bank Transfer', date: '2023-03-01', amount: '200,000' },
]);

const reports = ref([
  { id: 1, date: '2023-01-01', url: '#' },
  { id: 2, date: '2023-02-01', url: '#' },
  { id: 3, date: '2023-03-01', url: '#' },
]);

// Hàm chọn tab
const selectTab = (tab) => {
  currentTab.value = tab;
};

</script>

<style scoped>
@import "/src/assets/css/admin.css";
body,
html {
  margin: 0;
  padding: 0;
  font-family: 'Arial', sans-serif;
}

.finance-management {
  background-color: #fff;
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  animation: fadeIn 1s ease-in-out;
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

h1 {
  text-align: center;
  margin-bottom: 20px;
  color: #333;
  font-size: 28px;
  font-weight: bold;
}

.tabs {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.tabs button {
  padding: 10px 20px;
  border: none;
  background-color: #3498DB;
  color: white;
  margin: 0 5px;
  cursor: pointer;
  border-radius: 4px;
  transition: background-color 0.3s ease;
}

.tabs button:hover {
  background-color: #2980B9;
}

.tabs button.active {
  background-color: #1ABC9C;
}

.tab-content {
  text-align: left;
  padding: 20px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.tab-content h2 {
  margin-bottom: 10px;
}

.revenue-summary {
  background-color: #e0f7fa;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
  margin-bottom: 20px;
}

.total-revenue {
  font-size: 24px;
  color: #00796b;
}

.transaction-list,
.payment-list,
.report-list {
  list-style-type: none;
  padding: 0;
}

.transaction-list li,
.payment-list li,
.report-list li {
  padding: 10px;
  border-bottom: 1px solid #ddd;
}

.report-list a {
  color: #3498db;
  text-decoration: none;
}

.report-list a:hover {
  text-decoration: underline;
}
</style>
