<script setup>
import { onMounted, ref, watch, watchEffect } from 'vue'
import Message from './Message.vue'
import axios from 'axios'
import { ElScrollbar } from 'element-plus';
import { computed } from '@vue/reactivity';

let username = ref('')
let user_uuid = ref('')

let api_host = `http://${window.location.hostname}:5000`

let msg_list = ref([])

let msg_input = ref('')
let username_input = ref('')
let logined = ref(false)
let auto_scroll = ref(false)


async function user_login() {
  try {
    const response = await axios.post(api_host + "/gnchat/api/login", {
      "username": username_input.value
    })
    console.log(response)

    if (response.data.code == '0001') {
      alert('请检查用户名的输入是否合法')
    }
    if (response.data.code == '0000') {
      username.value = response.data.data.username
      user_uuid.value = response.data.data.user_uuid
      logined.value = true
    }

  } catch (error) {
    console.error(error)
  }
}

function format_message(msgs) {
  msgs.forEach((v, index) =>
    msgs[index].is_self = (msgs[index].owner_uuid == user_uuid.value ? true : false)
  )
  console.log(msgs)
  return msgs
}

function encrypt(data) {
  let str = JSON.stringify(data)
  let utf8Encode = new TextEncoder()
  let array = utf8Encode.encode(str)
  let res = []
  array.forEach((v, index) => res.push(v + 1))
  return { "data": res }
}

function decrypt(data) {
  data.forEach((v, index) => data[index]--)
  const bytesString = String.fromCharCode(...data)
  // console.log(bytesString)
  return JSON.parse(bytesString)
}

async function send_message() {
  try {
    const response = await axios.post(api_host + "/gnchat/api/pmsg",
      encrypt({
        "user_uuid": user_uuid.value,
        "content": msg_input.value
      })
    )

    console.log(response)

    if (response.data.code == '0000') {
      console.log("msg send ok")

      msg_input.value = ""
    }
  } catch (error) {
    console.error(error)
  }
}

const scrollbarRef = ref()

async function get_message() {
  try {
    const response = await axios.post(api_host + "/gnchat/api/gmsg",
      encrypt({
        "user_uuid": user_uuid.value,
      })
    )
    console.log(response)

    if (response.data.code == '0000') {
      const data = decrypt(response.data.data)
      let msg = data.reverse()
      msg = format_message(msg)
      msg_list.value = [...msg_list.value.concat(msg)]

    } else {
      console.error(response)
    }
  } catch (error) {
    console.error(error)
  }
}


async function get_history_message() {
  try {
    const response = await axios.post(api_host + "/gnchat/api/gmsg", encrypt({
      "user_uuid": user_uuid.value,
      "count": 20,
      "msg_time": msg_list.value.length == 0 ? (new Date()).valueOf() / 1000 : msg_list.value[0].send_time
    }))

    if (response.data.code == '0000') {
      const data = decrypt(response.data.data)
      let msg = data.reverse()
      msg = format_message(msg)
      msg_list.value = [...msg.concat(msg_list.value)]

    } else {
      console.error(response)
    }
  } catch (error) {
    console.log(error)
  }
}

const scrollbarViewHeight = computed(() => {
  return scrollbarRef.value.$el.firstChild.firstChild.clientHeight
})

// const stop = watchEffect(() => logined.value? setInterval(get_message, 1000): null)
// onMounted(() => logined.value? stop(): null)

// 自动滚动到最下面
watchEffect(() => {
  if (auto_scroll.value) {
    // console.log(scrollbarViewHeight)
    scrollbarRef.value.setScrollTop(scrollbarViewHeight.value)
  }
})

</script>

<template>
  <el-header class="header">
    <h1>GN 聊天室</h1>
    <div v-if="logined">
      欢迎你！{{ username }}
    </div>

    <el-row v-if="!logined">
      <el-col :span="5">
        用户名：
        <el-input :disabled="logined" v-model="username_input" placeholder="请输入用户名" />
      </el-col>
      <el-col>
        <button :disabled="logined" type="default" @click.stop="user_login"> 登录 </button>
      </el-col>
    </el-row>

  </el-header>

  <el-main v-show="logined">

    <el-row class="msg-container" sytle="width: 80%">
      <el-col sytle="width: 100%">
        <el-scrollbar height="500px" ref="scrollbarRef">
          <el-row v-for="msg of msg_list" style="padding-bottom: 10px">
            <Message :name="msg.owner_name" :msg-body="msg.content" :msg-type="msg.is_picture" :is-self="msg.is_self" />
          </el-row>
        </el-scrollbar>
      </el-col>
    </el-row>

    <el-row>
      <el-col :span="8">
        <el-input @keyup.enter.native.stop="send_message" v-model="msg_input" clearable />
      </el-col>

      <el-col :span="2">
        <el-button type="primary" @click.stop="send_message"> 发送 </el-button>
      </el-col>

    </el-row>

    <el-row>
      <el-button type="default" @click.stop="get_history_message"> 获取历史消息 </el-button>
    </el-row>
    <el-row>
      <el-switch 
        v-model="auto_scroll"
      />  自动滚动
    </el-row>

  </el-main>
</template>

<style scoped>
.header .h1 {
  font-weight: bold;
}

.msg-container {
  width: 80%
}
</style>
