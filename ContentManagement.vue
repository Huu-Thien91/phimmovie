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
    <div class="content-management">
      <h1>Quản lý Nội Dung</h1>
      <div class="tabs">
        <button @click="selectTab('articles')" :class="{ active: currentTab === 'articles' }">Bài viết và Đánh
          giá</button>
        <button @click="selectTab('comments')" :class="{ active: currentTab === 'comments' }">Quản lý Bình luận</button>
      </div>
      <!-- Quản lý bài viết -->
      <div v-if="currentTab === 'articles'" class="tab-content">
        <h2>Quản lý Bài viết và Đánh giá</h2>
        <form @submit.prevent="submitArticle" class="form">
          <div class="form-group">
            <label for="title">Tiêu đề bài viết</label>
            <input type="text" id="title" v-model="article.title" placeholder="Nhập tiêu đề..." />
          </div>
          <div class="form-group">
            <label for="content">Nội dung</label>
            <textarea id="content" v-model="article.content"
              placeholder="Nhập nội dung bài viết hoặc đánh giá..."></textarea>
          </div>
          <button type="submit" class="submit-button">Lưu Bài viết</button>
        </form>
        <div class="article-list">
          <h3>Danh sách Bài viết</h3>
          <ul>
            <li v-for="(article, index) in articles" :key="index">
              <h4>{{ article.title }}</h4>
              <p>{{ article.content }}</p>
              <button @click="editArticle(index)" class="edit-button">Sửa</button>
              <button @click="deleteArticle(index)" class="delete-button">Xóa</button>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <!-- Quản lý bình luận -->
    <div v-if="currentTab === 'comments'" class="tab-content">
      <h2>Quản lý Bình luận</h2>
      <ul class="comment-list">
        <li v-for="(comment, index) in comments" :key="index">
          <p>{{ comment.text }}</p>
          <small>Người dùng: {{ comment.user }}</small>
          <button @click="deleteComment(index)" class="delete-button">Xóa</button>
          <button @click="editComment(index)" class="edit-button">Sửa</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';


// Tab hiện tại
const currentTab = ref('articles');

// Dữ liệu bài viết
const articles = ref([
  { title: 'Bài viết 1', content: 'Nội dung bài viết 1...' },
  { title: 'Bài viết 2', content: 'Nội dung bài viết 2...' },
]);

const article = ref({ title: '', content: '' });

// Dữ liệu bình luận
const comments = ref([
  { text: 'Bình luận hay!', user: 'Người dùng 1' },
  { text: 'Đánh giá rất hữu ích.', user: 'Người dùng 2' },
]);

// Hàm chọn tab
const selectTab = (tab) => {
  currentTab.value = tab;
};

// Hàm thêm/sửa bài viết
const submitArticle = () => {
  if (article.value.editingIndex !== undefined) {
    articles.value[article.value.editingIndex] = { ...article.value };
    delete article.value.editingIndex;
  } else {
    articles.value.push({ ...article.value });
  }
  article.value = { title: '', content: '' };
};

// Hàm sửa bài viết
const editArticle = (index) => {
  article.value = { ...articles.value[index], editingIndex: index };
};

// Hàm xóa bài viết
const deleteArticle = (index) => {
  articles.value.splice(index, 1);
};

// Hàm sửa bình luận
const editComment = (index) => {
  const newText = prompt('Sửa bình luận:', comments.value[index].text);
  if (newText) comments.value[index].text = newText;
};

// Hàm xóa bình luận
const deleteComment = (index) => {
  comments.value.splice(index, 1);
};
</script>

<style scoped>
@import "/src/assets/css/admin.css";
body {
  font-family: 'Arial', sans-serif;
  background-color: #f9f9f9;
  margin: 0;
}
.container {
    display: flex;
    height: 100vh; /* Chiều cao toàn màn hình */
  }
.content-management {
  flex-grow: 1;
  margin: left 100%; /* Khoảng cách bù cho thanh sidebar */
  padding: 25px;
  background-color: #f8f9fa;
  overflow: auto;
  animation: fadeIn 1s ease-in-out;
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

.form {
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  font-weight: bold;
}

input[type="text"],
textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
  transition: border-color 0.3s ease;
}

input[type="text"]:focus,
textarea:focus {
  border-color: #4CAF50;
  outline: none;
}

.submit-button {
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.submit-button:hover {
  background-color: #45a049;
}

.article-list ul,
.comment-list {
  list-style: none;
  padding: 0;
}

.article-list li,
.comment-list li {
  padding: 10px;
  border-bottom: 1px solid #ddd;
  margin-bottom: 10px;
  background-color: #f9f9f9;
  border-radius: 4px;
}

.edit-button {
  padding: 5px 10px;
  background-color: #3498DB;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 5px;
}

.edit-button:hover {
  background-color: #2980B9;
}

.delete-button {
  padding: 5px 10px;
  background-color: #E74C3C;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.delete-button:hover {
  background-color: #C0392B;
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
</style>
