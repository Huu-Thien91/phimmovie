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
    <div class="vip-payment-management">
      <h1>Quản lí Tài Khoản VIP và Thanh Toán</h1>

      <div class="tabs">
        <button @click="selectTab('vipPackages')" :class="{ active: currentTab === 'vipPackages' }">Quản lý Gói
          VIP</button>
        <button @click="selectTab('paymentHistory')" :class="{ active: currentTab === 'paymentHistory' }">Lịch sử Thanh
          Toán</button>
        <button @click="selectTab('paymentGateways')" :class="{ active: currentTab === 'paymentGateways' }">Cổng Thanh
          Toán</button>
      </div>

      <!-- Quản lý Gói VIP -->
      <div v-if="currentTab === 'vipPackages'" class="tab-content">
        <h2>Quản lý Gói VIP</h2>
        <form @submit.prevent="addVipPackage">
          <div class="form-group">
            <label for="packageName">Tên Gói</label>
            <input type="text" id="packageName" v-model="newPackage.name" placeholder="Nhập tên gói VIP" />
          </div>
          <div class="form-group">
            <label for="price">Giá</label>
            <input type="number" id="price" v-model="newPackage.price" placeholder="Nhập giá gói VIP" />
          </div>
          <div class="form-group">
            <label for="benefits">Quyền lợi</label>
            <textarea id="benefits" v-model="newPackage.benefits" placeholder="Nhập quyền lợi của gói VIP"></textarea>
          </div>
          <button type="submit" class="save-button">Thêm Gói VIP</button>
        </form>

        <h3>Danh sách Gói VIP</h3>
        <table class="package-table">
          <thead>
            <tr>
              <th>Tên Gói</th>
              <th>Giá</th>
              <th>Quyền Lợi</th>
              <th>Hành Động</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(pkg, index) in vipPackages" :key="index">
              <td>{{ pkg.name }}</td>
              <td>{{ pkg.price }} VND</td>
              <td>{{ pkg.benefits }}</td>
              <td>
                <button @click="editVipPackage(index)" class="edit-button">Sửa</button>
                <button @click="deleteVipPackage(index)" class="delete-button">Xóa</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Lịch sử Thanh Toán -->
      <div v-if="currentTab === 'paymentHistory'" class="tab-content">
        <h2>Lịch sử Thanh Toán</h2>
        <table class="history-table">
          <thead>
            <tr>
              <th>Tên Người Dùng</th>
              <th>Gói VIP</th>
              <th>Số Tiền</th>
              <th>Thời Gian Thanh Toán</th>
              <th>Phương Thức</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(payment, index) in paymentHistory" :key="index">
              <td>{{ payment.user }}</td>
              <td>{{ payment.package }}</td>
              <td>{{ payment.amount }} VND</td>
              <td>{{ payment.timestamp }}</td>
              <td>{{ payment.method }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Cổng Thanh Toán -->
      <div v-if="currentTab === 'paymentGateways'" class="tab-content">
        <h2>Cổng Thanh Toán</h2>
        <p>Hệ thống hỗ trợ các cổng thanh toán sau:</p>
        <ul>
          <li>Momo</li>
          <li>VNPay</li>
          <li>PayPal</li>
          <li>Thẻ tín dụng</li>
        </ul>
        <button class="activate-button" @click="activatePaymentGateway">Kích Hoạt Cổng Thanh Toán</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
const currentTab = ref('vipPackages');

const vipPackages = ref([]);
const newPackage = ref({ name: '', price: '', benefits: '' });

const paymentHistory = ref([
  { user: 'Nguyễn Văn A', package: 'Gói VIP Premium', amount: '500000', timestamp: '2023-03-01 12:30', method: 'Momo' },
  { user: 'Trần Thị B', package: 'Gói VIP Basic', amount: '200000', timestamp: '2023-03-02 15:00', method: 'VNPay' },
]);

// Chuyển tab
const selectTab = (tab) => {
  currentTab.value = tab;
};

// Thêm, sửa, xóa gói VIP
const addVipPackage = () => {
  vipPackages.value.push({ ...newPackage.value });
  newPackage.value = { name: '', price: '', benefits: '' };
};

const editVipPackage = (index) => {
  alert(`Chỉnh sửa gói VIP: ${vipPackages.value[index].name}`);
};

const deleteVipPackage = (index) => {
  vipPackages.value.splice(index, 1);
  alert('Gói VIP đã được xóa thành công!');
};

// Kích hoạt cổng thanh toán
const activatePaymentGateway = () => {
  alert('Cổng thanh toán đã được kích hoạt!');
};
</script>

<style scoped>
@import "/src/assets/css/admin.css";

.vip-payment-management {
  background-color: #fff;
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  animation: fadeIn 1s ease-in-out;
}

h1 {
  text-align: center;
  color: #34495e;
  margin-bottom: 20px;
}

.tabs {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
  gap: 10px;
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

.tabs button {
  padding: 10px 20px;
  font-size: 14px;
  border: none;
  border-radius: 6px;
  background-color: #3498db;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.tabs button:hover {
  background-color: #2980b9;
}

.tabs button.active {
  background-color: #1abc9c;
}

.tab-content {
  padding: 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  font-weight: bold;
  margin-bottom: 5px;
}

input,
textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 10px 15px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  color: white;
  cursor: pointer;
}

.save-button,
.add-button {
  background-color: #4CAF50;
}

.edit-button {
  background-color: #2980b9;
}

.delete-button {
  background-color: #e74c3c;
}

.activate-button {
  background-color: #f39c12;
}
</style>
