<script setup>
import { inject, ref, reactive, onMounted } from "vue"
import socketManager from '../socketManager.js'


// #region global state
const userName = inject("userName")
// #endregion

// #region local variable
const socket = socketManager.getInstance()
// #endregion

// #region reactive variable
const chatContent = ref("")
const chatList = reactive([])
// #endregion

// #region lifecycle
onMounted(() => {
  registerSocketEvent()
})
// #endregion

// #region browser event handler
// 投稿メッセージをサーバに送信する
const onPublish = () => {
  // 投稿メッセージをサーバに送信
  const message = `${userName.value}さん: ${chatContent.value}`
  socket.emit("publishEvent", message)

  // 入力欄を初期化
  chatContent.value = ""
}

// 退室メッセージをサーバに送信する
const onExit = () => {

}

// メモを画面上に表示する
const onMemo = () => {
  const message = `${userName.value}さんのメモ: ${chatContent.value}`
  chatList.unshift(message)

  // 入力欄を初期化
  chatContent.value = ""

}
// #endregion

// #region socket event handler
// サーバから受信した入室メッセージ画面上に表示する
const onReceiveEnter = (data) => {
  chatList.push(data)
}

// サーバから受信した退室メッセージを受け取り画面上に表示する
const onReceiveExit = (data) => {
  chatList.push(data)
}

// サーバから受信した投稿メッセージを画面上に表示する
const onReceivePublish = (data) => {

  chatList.push(data)
}
// #endregion

// #region local methods
// イベント登録をまとめる
const registerSocketEvent = () => {
  // 入室イベントを受け取ったら実行
  socket.on("enterEvent", (data) => {
    onReceiveEnter(data)
  })

  // 退室イベントを受け取ったら実行
  socket.on("exitEvent", (data) => {
    onReceiveExit(data)
  })

  // 投稿イベントを受け取ったら実行
  socket.on("publishEvent", (data) => {
    onReceivePublish(data)
  })
}

// メッセージがメモかどうかを判定する
const isMemo = (chat) => {
  return chat.isMemo
}

// #endregion
</script>


<template>
  <div class="mx-auto my-5 px-4">
    <h1 class="text-h3 font-weight-medium">Vue.js Chat チャットルーム</h1>
    <div class="mt-10">
      <p>ログインユーザ：{{ userName }}さん</p>
      <textarea v-model="chatContent" variant="outlined" placeholder="投稿文を入力してください" rows="4" class="area"></textarea>
      <div class="mt-5">
        <button class="button-normal" @click="onPublish">投稿</button>
        <button class="button-normal util-ml-8px" @click="onMemo">メモ</button>
      </div>
      <div class="mt-5" v-if="chatList.length !== 0">
        <ul>
          <li class="item mt-4" v-for="(chat, i) in chatList" :key="i" :class="{publish: chat.startsWith(userName + 'さん:'), memo: chat.startsWith(userName + 'さんのメモ:') }">
            {{ chat }}
          </li>
        </ul>
      </div>
    </div>
    <router-link to="/" class="link">
      <button type="button" class="button-normal button-exit" @click="onExit">退室する</button>
    </router-link>
  </div>
</template>

<style scoped>
.publish {
  color: red !important;
  font-style: italic !important;
  background-color: #f8f9fa !important;
  padding: 5px !important;
  border-left: 4px solid #ced4da !important;
  margin-top: 10px !important;
  
}

.memo {
  color: blue; /* 青色のテキスト */
}

.link {
  text-decoration: none;
}

.area {
  width: 500px;
  border: 1px solid #000;
  margin-top: 8px;
}

.item {
  display: block;
}

.util-ml-8px {
  margin-left: 8px;
}

.button-exit {
  color: #000;
  margin-top: 8px;
}
</style>
