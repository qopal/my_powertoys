<template>
  <div class="timeline">
    <h1 class="center">勤務記録</h1>
    <div class="center">
      <table>
        <thead>
          <tr>
            <th>開始時間</th>
            <th>終了時間</th>
            <th>作業名</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(timeLog, index) in timeLogs" :key="index">
            <td>{{ timeLog.startTime }}</td>
            <td>{{ timeLog.endTime }}</td>
            <td>{{ timeLog.projectName }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="center">
      <button @click="clear">ログをクリア</button>
    </div>
  </div>
  
</template>

<script>
export default {
  name: 'TimeLine',
  data() {
    return {
      timeLogs: []
    }
  },
  created() {
    // ブラウザのローカルストレージからデータを取得
    try {
      const storedLogs = localStorage.getItem('timeLogs');
      if (storedLogs) {
        this.timeLogs = JSON.parse(storedLogs);
      }
    } catch (error) {
      console.error('ローカルストレージからの読み込みに失敗しました:', error);
    }
  },
  watch: {
    timeLogs: {
      handler: function (timeLogs) {
        localStorage.setItem('timeLogs', JSON.stringify(timeLogs));
      },
      deep: true
    }
  },
  methods: {
    clear() {
      if (confirm('勤務記録のログをクリアしますか？')) {
        this.timeLogs = []
      }
    }
  }
}
</script>

<style>
.timeline {
  border: 2px solid #000; /* 黒色の外枠 */
  border-radius: 15px; /* 枠を丸くする */
  background-color: #98d9aa;
  padding: 20px;
  margin: 20px;
}

table {
  width: 100%;
  border-collapse: collapse; /* セルの境界線を一本にする */
  box-shadow: 0 2px 15px rgba(0,0,0,0.1); /* テーブルに影をつける */
}

th, td {
  padding: 10px 20px;
  text-align: left;
  border-bottom: 1px solid #ddd; /* セルの下線 */
}

th {
  background-color: #4CAF50;
  color: white;
}

tr:nth-child(even) {
  background-color: #f2f2f2;
}

.center {
  text-align: center;
  font-size: 16px;
}
</style>
