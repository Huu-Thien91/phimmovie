<template>
  <div class="overview-dashboard">
    <h1>Tổng Quan</h1>
    <div class="statistics-cards">
      <!-- Số liệu thống kê chính -->
      <div class="card" v-for="stat in statistics" :key="stat.title">
        <h3>{{ stat.title }}</h3>
        <p>{{ stat.value }}</p>
        <small>{{ stat.description }}</small>
      </div>
    </div>

    <div class="charts-section">
      <!-- Biểu đồ lượt xem -->
      <div class="chart">
        <h2>Lượt Xem Theo Thời Gian</h2>
        <canvas id="viewChart"></canvas>
      </div>

      <!-- Biểu đồ doanh thu -->
      <div class="chart">
        <h2>Doanh Thu Theo Tháng</h2>
        <canvas id="revenueChart"></canvas>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Chart from 'chart.js/auto';

// Dữ liệu thống kê chính
const statistics = ref([
  { title: 'Tổng Người Dùng', value: 10000, description: 'Người dùng đã đăng ký' },
  { title: 'Người Dùng VIP', value: 1500, description: 'Người dùng VIP đang hoạt động' },
  { title: 'Số Lượng Phim', value: 500, description: 'Phim có sẵn trong hệ thống' },
  { title: 'Tổng Lượt Xem', value: 200000, description: 'Lượt xem toàn hệ thống' },
  { title: 'Đăng Ký Mới', value: 50, description: 'Hôm nay' },
  { title: 'Doanh Thu Tháng', value: '100,000,000 VND', description: 'Trong tháng hiện tại' },
]);

// Tạo biểu đồ
const createCharts = () => {
  const viewCtx = document.getElementById('viewChart').getContext('2d');
  const revenueCtx = document.getElementById('revenueChart').getContext('2d');

  // Biểu đồ lượt xem
  new Chart(viewCtx, {
    type: 'line',
    data: {
      labels: ['Tháng 1', 'Tháng 2', 'Tháng 3', 'Tháng 4', 'Tháng 5'],
      datasets: [
        {
          label: 'Lượt Xem',
          data: [5000, 10000, 15000, 20000, 25000],
          borderColor: '#3498db',
          backgroundColor: 'rgba(52, 152, 219, 0.2)',
          fill: true,
        },
      ],
    },
    options: {
      responsive: true,
      plugins: { legend: { display: true } },
    },
  });

  // Biểu đồ doanh thu
  new Chart(revenueCtx, {
    type: 'bar',
    data: {
      labels: ['Tháng 1', 'Tháng 2', 'Tháng 3', 'Tháng 4', 'Tháng 5'],
      datasets: [
        {
          label: 'Doanh Thu (VND)',
          data: [20000000, 30000000, 40000000, 50000000, 60000000],
          backgroundColor: '#1abc9c',
        },
      ],
    },
    options: {
      responsive: true,
      plugins: { legend: { display: true } },
    },
  });
};

// Khởi tạo biểu đồ khi component được tải
onMounted(() => {
  createCharts();
});
</script>

<style scoped>
body {
  font-family: 'Arial', sans-serif;
  background-color: #f9f9f9;
  margin: 0;
}

.overview-dashboard {
  max-width: 1200px;
  margin: 20px auto;
  padding: 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  font-size: 28px;
  color: #333;
  margin-bottom: 20px;
}

.statistics-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  margin-bottom: 40px;
}

.card {
  padding: 20px;
  background: #f4f6f9;
  border: 1px solid #ddd;
  border-radius: 8px;
  text-align: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.card h3 {
  font-size: 18px;
  margin-bottom: 10px;
  color: #2c3e50;
}

.card p {
  font-size: 22px;
  font-weight: bold;
  color: #3498db;
}

.card small {
  font-size: 12px;
  color: #7f8c8d;
}

.charts-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 40px;
}

.chart {
  padding: 20px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.chart h2 {
  font-size: 20px;
  margin-bottom: 20px;
  color: #34495e;
}
</style>
