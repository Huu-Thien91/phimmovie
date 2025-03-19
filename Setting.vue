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
    <div class="settings-container">
      <!-- Thanh Tab chuyển đổi giữa các phần cài đặt -->
      <div class="settings-tabs">
        <div class="settings-tab" :class="{ active: activeTab === 'account' }" @click="activeTab = 'account'">
          Tài khoản Admin
        </div>
        <div class="settings-tab" :class="{ active: activeTab === 'appearance' }" @click="activeTab = 'appearance'">
          Giao diện
        </div>
        <div class="settings-tab" :class="{ active: activeTab === 'security' }" @click="activeTab = 'security'">
          Bảo mật
        </div>
        <div class="settings-tab" :class="{ active: activeTab === 'payments' }" @click="activeTab = 'payments'">
          Thanh toán & Gói đăng ký
        </div>
        <div class="settings-tab" :class="{ active: activeTab === 'streaming' }" @click="activeTab = 'streaming'">
          Phát trực tuyến
        </div>
        <div class="settings-tab" :class="{ active: activeTab === 'system' }" @click="activeTab = 'system'">
          Hệ thống
        </div>
      </div>

      <!-- Nội dung các tab cài đặt -->
      <div class="tab-content">
        <!-- Tab: Tài khoản Admin -->
        <div v-if="activeTab === 'account'">
          <label for="name">Tên Admin:</label>
          <input v-model="settings.account.name" id="name" type="text" />

          <label for="email">Email:</label>
          <input v-model="settings.account.email" id="email" type="email" />

          <label for="phone">Số điện thoại:</label>
          <input v-model="settings.account.phone" id="phone" type="text" />

          <label for="password">Mật khẩu:</label>
          <input v-model="settings.account.password" id="password" type="password" />

          <label for="role">Vai trò:</label>
          <select v-model="settings.account.role" id="role">
            <option value="Super Admin">Super Admin</option>
            <option value="Moderator">Moderator</option>
            <option value="Editor">Editor</option>
          </select>

          <div class="switch-container">
            <label>Bật 2FA</label>
            <input v-model="settings.account.enable2FA" type="checkbox" />
          </div>
        </div>

        <!-- Tab: Giao diện -->
        <div v-if="activeTab === 'appearance'">
          <label for="logo">Logo:</label>
          <input @change="handleFileChange('logo', $event)" id="logo" type="file" />

          <label for="favicon">Favicon:</label>
          <input @change="handleFileChange('favicon', $event)" id="favicon" type="file" />

          <label for="theme">Chủ đề giao diện:</label>
          <select v-model="settings.appearance.theme" id="theme">
            <option value="light">Sáng</option>
            <option value="dark">Tối</option>
          </select>

          <label for="layout">Bố cục:</label>
          <select v-model="settings.appearance.layout" id="layout">
            <option value="grid">Lưới</option>
            <option value="list">Danh sách</option>
          </select>
        </div>

        <!-- Tab: Bảo mật -->
        <div v-if="activeTab === 'security'">
          <label for="passwordPolicy">Chính sách mật khẩu:</label>
          <textarea v-model="settings.security.passwordPolicy" id="passwordPolicy"></textarea>

          <div class="switch-container">
            <label>Chặn tài khoản vi phạm</label>
            <input v-model="settings.security.blockViolation" type="checkbox" />
          </div>
        </div>

        <!-- Tab: Thanh toán -->
        <div v-if="activeTab === 'payments'">
          <label for="paymentGateway">Cổng thanh toán:</label>
          <select v-model="settings.payments.paymentGateway" id="paymentGateway">
            <option value="Momo">Momo</option>
            <option value="VNPay">VNPay</option>
            <option value="PayPal">PayPal</option>
          </select>

          <label for="subscriptionDuration">Thời gian hết hạn gói đăng ký:</label>
          <input v-model="settings.payments.subscriptionDuration" id="subscriptionDuration" type="number" />
        </div>

        <!-- Tab: Phát trực tuyến -->
        <div v-if="activeTab === 'streaming'">
          <label for="videoQuality">Chất lượng video:</label>
          <select v-model="settings.streaming.videoQuality" id="videoQuality">
            <option value="480p">480p</option>
            <option value="720p">720p</option>
            <option value="1080p">1080p</option>
            <option value="4K">4K</option>
          </select>

          <div class="switch-container">
            <label>Hỗ trợ phụ đề & Thuyết minh</label>
            <input v-model="settings.streaming.subtitles" type="checkbox" />
          </div>
        </div>

        <!-- Tab: Hệ thống -->
        <div v-if="activeTab === 'system'">
          <div class="switch-container">
            <label>Bảo trì hệ thống</label>
            <input v-model="settings.system.maintenanceMode" type="checkbox" />
          </div>

          <label for="seoMetaTitle">Meta Title (SEO):</label>
          <input v-model="settings.system.seoMetaTitle" id="seoMetaTitle" type="text" />

          <label for="seoMetaDescription">Meta Description (SEO):</label>
          <textarea v-model="settings.system.seoMetaDescription" id="seoMetaDescription"></textarea>
        </div>

        <button @click="saveSettings" class="save-btn">Lưu Cài Đặt</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      activeTab: 'account', // Tab mặc định là tài khoản admin
      settings: {
        account: {
          name: '',
          email: '',
          phone: '',
          password: '',
          role: 'Super Admin',
          enable2FA: false,
        },
        appearance: {
          logo: null,
          favicon: null,
          theme: 'light',
          layout: 'grid',
        },
        security: {
          passwordPolicy: '',
          blockViolation: false,
        },
        payments: {
          paymentGateway: 'Momo',
          subscriptionDuration: 12, // tháng
        },
        streaming: {
          videoQuality: '1080p',
          subtitles: false,
        },
        system: {
          maintenanceMode: false,
          seoMetaTitle: '',
          seoMetaDescription: '',
        },
      },
    };
  },
  methods: {
    // Xử lý thay đổi file
    handleFileChange(field, event) {
      const file = event.target.files[0];
      if (file) {
        this.settings.appearance[field] = file;
      }
    },
    goBack() {
      this.$router.go(-1);
    },
    saveSettings() {
      // Xử lý lưu cài đặt ở đây (ví dụ: gọi API để lưu lên server)
      console.log('Settings saved:', this.settings);
    },
  },
};
</script>

<style scoped>
@import "/src/assets/css/admin.css";
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

/* Đặt background và padding chung cho toàn bộ trang */
.settings-container {
  background-color: #fff;
  border-radius: 10px;
  padding: 30px;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
  animation: fadeIn 1s ease-in-out;
}

/* Style cho các Tab */
.settings-tabs {
  display: flex;
  margin-bottom: 20px;
}

.settings-tab {
  padding: 10px 20px;
  background-color: #e3e3e3;
  margin-right: 10px;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
}

.settings-tab.active {
  background-color: #007bff;
  color: white;
}

/* Style cho mỗi phần bên trong các tab */
.tab-content {
  padding: 20px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.tab-content input,
.tab-content select,
.tab-content textarea {
  width: 100%;
  padding: 12px;
  margin-bottom: 15px;
  border-radius: 5px;
  border: 1px solid #ddd;
  font-size: 16px;
}

.switch-container {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.switch-container input {
  width: 40px;
  height: 20px;
  appearance: none;
  background-color: #ccc;
  border-radius: 50px;
  position: relative;
  cursor: pointer;
}

.switch-container input:checked {
  background-color: #007bff;
}

.switch-container input:before {
  content: '';
  width: 18px;
  height: 18px;
  background-color: white;
  border-radius: 50%;
  position: absolute;
  top: 1px;
  left: 1px;
  transition: left 0.3s ease;
}

.switch-container input:checked:before {
  left: 20px;
}

/* Style cho nút lưu */
.save-btn {
  background-color: #28a745;
  color: white;
  padding: 10px 20px;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
}

.save-btn:hover {
  background-color: #218838;
}
</style>
