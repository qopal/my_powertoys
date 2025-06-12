<template>
  <div class="center" style="padding: 0px 0px;">
    <!-- 左側のコンテンツ -->
    <div style="flex: 3;">
      <submitTime />
    </div>

    <!-- 右側のコンテンツ -->
    <div style="flex: 1;">
      <timeLine />
    </div>
  </div>
</template>

<script>
import submitTime from './components/submitTime.vue'
import timeLine from './components/timeLine.vue'
export default {
  name: 'OptionPage',
  components: {
    submitTime,
    timeLine
  }
}
</script>

<style>
@import '/src/assets/style.css';
</style>