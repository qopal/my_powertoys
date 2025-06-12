<template>
  <div v-if="showPopup" class="popup-overlay" @click.self="showPopup = false">
    <div class="popup-content">
      <!-- ここにポップアップで表示したいHTMLを追加 -->
      <h2>作業分類ID</h2>
      <p>
        Redmineの時間登録のページで開発者ツールから登録したい作業分類のIDを調べてください。<br>
        以下の画像の場合は、設計作業:8、開発作業:9です。
        <img :src="activityImage" alt="作業分類IDの調べ方" style="width: 100%;">
      </p>
      <button @click="showPopup = false">閉じる</button>
    </div>
  </div>

  <h1 class="center">勤務記録の集計</h1>
  <div class="center">
    <table>
      <thead>
        <tr>
          <th>作業名</th>
          <th>合計時間</th>
          <th>チケットID</th>
          <th>作業分類ID
            <button @click="showPopup = true">
              <i class="fa-solid fa-question"></i>
            </button>
          </th>
          <th>登録時コメント</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(project, index) in projects" :key="index"
          draggable="true"
          @dragstart="dragList($event,index)"
          @drop="dropList($event, index)"
          @dragover.prevent
          @dragenter.prevent
        >
          <!-- 作業名 -->
          <td>
            <div style="display: flex;">
              <a v-show="redmineUrl && project.redmineIssueId" :href="`${redmineUrl}/issues/${project.redmineIssueId}`" target="_blank">
                <img src="https://www.redmine.org/favicon.ico" alt="Redmineのアイコン" style="width: 1rem;">
              </a>
              <input type="text" v-model="project.name">
            </div>
          </td>
          <!-- 合計時間 -->
          <td><input type="time" class="time-input" :value="formatTimeForInput(project.sum)" @input="updateSum(project, $event.target.value)"></td>
          <!-- チケットID -->
          <td>
            <input type="text" v-model="project.redmineIssueId" style="width: 4rem;">
          </td>
          <!-- 作業分類ID -->
          <td>
            <input type="text" v-model="project.activityId" style="width: 4rem;">
          </td>
          <!-- 登録時コメント -->
          <td>
            <input type="text" v-model="project.comment">
          </td>
        </tr>
      </tbody>
    </table>
    <button style="width: 9rem;" @click="registerAllToRedmine">Redmineに一括登録</button>
  </div>

  <br>
  <div class="center">
    <button @click="apiKeyShow = !apiKeyShow">
      <span v-show="!apiKeyShow">接続情報登録</span>
      <span v-show="apiKeyShow">APIキーと接続先URLを非表示</span>
    </button>
    <div v-show="apiKeyShow">
      <input type="text" v-model="apiKey" placeholder="APIキーを入力" style="width: 300px;">
      <button @click="saveApiKey">保存</button>
      <br>
      <input type="text" v-model="redmineUrl" placeholder="RedmineのURLを入力" style="width: 300px;">
      <button @click="saveRedmineUrl">保存</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import activityImage from '../assets/activity.png' // 画像をインポート

export default {
  name: 'SubmitTime',
  data() {
    return {
      activityImage, // データとして追加
      projects: [],
      redmineUrl: '',
      apiKey: '',
      apiKeyShow: false,
      successProjects: [],
      failedProjects: [],
      deleteMode: false,
      showPopup: false
    }
  },
  created() {
    // ローカルストレージからプロジェクト情報を取得
    this.projects = JSON.parse(localStorage.getItem('projects')) || []
    this.apiKey = localStorage.getItem('apiKey') || ''
    this.redmineUrl = localStorage.getItem('redmineUrl') || ''
  },
  watch:{
    // プロジェクト情報が変更されたらローカルストレージに保存
    projects: {
      handler: function (projects) {
        localStorage.setItem('projects', JSON.stringify(projects))
      },
      deep: true
    }
  },
  computed: {
    // 登録内容の確認の為のメッセージ
    confirmMessage() {
      return this.projects.map(project => {
        console.log(project.sum)
        if (project.sum === '00:00:00') return
        if (!project.redmineIssueId) return
        if (!project.activityId) return
        return `${project.name} ${project.sum}時間`
      }).filter(Boolean).join('\n')
    }
  },
  methods: {
    registerAllToRedmine() {
      if (!confirm('以下の内容でRedmineに一括登録しますか？' + '\n' + this.confirmMessage)) return
      // 登録結果メッセージを初期化
      this.successProjects = [];
      this.failedProjects = [];

      const requests = this.projects.map(project => {
        const totalHours = this.getTotalHours(project);
        if (totalHours === null) return Promise.resolve();

        // Redmineに時間を登録
        return axios.post(`${this.redmineUrl}/time_entries.json`, {
          time_entry: {
            issue_id: project.redmineIssueId,
            hours: totalHours, // 変換した浮動小数点数を使用
            activity_id: project.activityId,
            comments: project.comment
          }
        }, {
          headers: {
            'Content-Type': 'application/json',
            'X-Redmine-API-Key': this.apiKey
          }
        })
        .then(response => {
          this.successProjects.push(`#${project.redmineIssueId} ${project.name}`);
          console.log('Success:', response.data.time_entry);
        })
        .catch(error => {
          this.failedProjects.push(`#${project.redmineIssueId} ${project.name}`);
          console.error('Error:', error);
        });
      });

      Promise.all(requests)
        .then(() => {
          let successMessage = '🟢以下のプロジェクトの登録に成功しました\n';
          successMessage += this.successProjects.join('\n');
          let failedMessage = '🔴以下のプロジェクトの登録に失敗しました\n';
          failedMessage += this.failedProjects.join('\n');
          alert(successMessage); // すべてのリクエストが完了した後にアラートを表示
          alert(failedMessage); // すべてのリクエストが完了した後にアラートを表示
        });
    },
    saveApiKey() {
      localStorage.setItem('apiKey', this.apiKey)
      alert('APIキーを保存しました')
    },
    saveRedmineUrl() {
      localStorage.setItem('redmineUrl', this.redmineUrl)
      alert('RedmineのURLを保存しました')
    },
    formatTimeForInput(timeString) {
      // "H:MM:SS"または"H:MM"形式の時間を"HH:MM"に変換する
      const parts = timeString.split(':')
      let hours = parts[0]
      let minutes = parts[1]

      // 時間が一桁の場合はゼロを追加
      if (hours.length === 1) {
        hours = '0' + hours
      }
      // 秒を無視して"HH:MM"形式を返す
      return `${hours}:${minutes}`
    },
    updateSum(project, newValue) {
      // newValueは"HH:MM"形式で受け取る
      // 必要に応じて秒を追加してプロジェクトのsumに設定
      project.sum = newValue + ':00'
    },
    getTotalHours(project) {
      // project.sumから時間と分を分離
      const [hours, minutes] = project.sum.split(':').map(Number);
      // 分を時間に変換し、時間に加算
      const totalHours = hours + minutes / 60;
      let message = '';
      if (totalHours === 0) message += '合計時間が0 ';
      if (!project.redmineIssueId) message += 'チケットIDが未入力 ';
      if (!project.activityId) message += '作業分類IDが未入力 ';
      if (message) {
        this.failedProjects.push(`#${project.redmineIssueId} ${project.name}`+ '：'+message)
        return null
      }
      return totalHours
    },
    dragList(event, dragIndex) {
      event.dataTransfer.effectAllowed = 'move'
      event.dataTransfer.dropEffect = 'move'
      event.dataTransfer.setData('drag-index',dragIndex)
    },
    dropList(event, dropIndex) {
      const dragIndex = event.dataTransfer.getData('drag-index')
      const deleteList = this.projects.splice(dragIndex,1)
      this.projects.splice(dropIndex, 0, deleteList[0])
    }
  }
}
</script>

<style>
input {
  width: 100%;
  font-size: 16px;
}

.popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.popup-content {
  background: white;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}
</style>
