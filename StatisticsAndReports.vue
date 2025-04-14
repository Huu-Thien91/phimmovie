<template>
  <div class="container">
    <h1>Danh Sách Phim Bộ</h1>

    <table class="table">
      <thead>
        <tr>
          <th>Avatar</th>
          <th>Tiêu đề</th>
          <th>Hành động</th>
        </tr>
      </thead>
      <tbody>
        <template v-for="series in seriesList" :key="series.seriesId">
          <!-- Dòng chính hiển thị phim -->
          <tr @click="toggleEpisodes(series.seriesId)">
            <td>
              <img :src="series.avatarUrl" alt="avatar" class="avatar" />
            </td>
            <td>{{ series.title }}</td>
            <td>
              <button class="btn btn-info">Xem tập</button>
            </td>
          </tr>

          <!-- Dòng phụ hiển thị danh sách tập nếu được mở rộng -->
          <tr v-if="expandedSeriesId === series.seriesId">
            <td colspan="3">
              <h3>Danh sách tập phim</h3>
              <button @click.stop="addEpisode(series.seriesId)" class="btn btn-primary">Thêm tập</button>
              <table class="table inner-table">
                <thead>
                  <tr>
                    <th>#</th>
                    <th>Tiêu đề</th>
                    <th>Hành động</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(ep, i) in series.episodes" :key="ep.episodeId">
                    <td>{{ i + 1 }}</td>
                    <td>{{ ep.title }}</td>
                    <td>
                      <button class="btn btn-warning" @click.stop="editEpisode(series.seriesId, ep.episodeId)">Sửa</button>
                      <button class="btn btn-danger" @click.stop="deleteEpisode(series.seriesId, ep.episodeId)">Xoá</button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      expandedSeriesId: null,
      seriesList: [
        {
          seriesId: 1,
          title: "Phim Hành Động",
          avatarUrl: "https://via.placeholder.com/80",
          episodes: [
            { episodeId: 1, title: "Tập 1" },
            { episodeId: 2, title: "Tập 2" },
          ],
        },
        {
          seriesId: 2,
          title: "Phim Hài",
          avatarUrl: "https://via.placeholder.com/80",
          episodes: [],
        },
        {
          seriesId: 3,
          title: "Phim Lãng Mạn",
          avatarUrl: "https://via.placeholder.com/80",
          episodes: [
            { episodeId: 3, title: "Tập 1" },
          ],
        },
      ],
    };
  },
  methods: {
    toggleEpisodes(seriesId) {
      this.expandedSeriesId = this.expandedSeriesId === seriesId ? null : seriesId;
    },
    addEpisode(seriesId) {
      const series = this.seriesList.find(s => s.seriesId === seriesId);
      const newId = Date.now();
      series.episodes.push({
        episodeId: newId,
        title: "Tập mới",
      });
    },
    editEpisode(seriesId, episodeId) {
      const episode = this.seriesList
        .find(s => s.seriesId === seriesId)
        .episodes.find(e => e.episodeId === episodeId);
      const newTitle = prompt("Sửa tiêu đề tập:", episode.title);
      if (newTitle) episode.title = newTitle;
    },
    deleteEpisode(seriesId, episodeId) {
      const series = this.seriesList.find(s => s.seriesId === seriesId);
      series.episodes = series.episodes.filter(e => e.episodeId !== episodeId);
    },
  },
};
</script>

<style scoped>
.container {
  padding: 20px;
}

.avatar {
  width: 60px;
  height: 60px;
  border-radius: 8px;
  cursor: pointer;
}

.table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.table th,
.table td {
  padding: 10px;
  border: 1px solid #ccc;
}

.inner-table {
  background: #f9f9f9;
  margin-top: 10px;
}

.btn {
  padding: 5px 10px;
  font-size: 12px;
  margin: 2px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-info {
  background: #17a2b8;
  color: white;
}

.btn-primary {
  background: #007bff;
  color: white;
}

.btn-warning {
  background: #ffc107;
  color: black;
}

.btn-danger {
  background: #dc3545;
  color: white;
}
</style>
