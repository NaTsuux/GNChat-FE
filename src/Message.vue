<script setup>
defineProps(['name', 'msgBody', 'msgType', 'isSelf'])
</script>

<template>
  <div class="message">
    <el-row :class="['msg-sender', isSelf ? 'msg-self' : '']">
      {{ name }}
    </el-row>
    <el-row :class="['msg-body', isSelf ? 'msg-self': '']">
      {{ msgBody }}
    </el-row>
  </div>
</template>

<style scoped>
.message {
  margin-bottom: 5px;
  padding-right: 20px;
  margin: 0 auto;
  width: 100%;
  height: fit-content;
}

.msg-self {
  text-align: right;
  justify-content: flex-end;
}

.message .msg-sender {
  font-size: 12px;
}

.message .msg-body {
  font-size: 17px;
  /* border-radius: 10px; */
}
</style>