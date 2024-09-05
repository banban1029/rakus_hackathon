<script setup>
import { inject, ref, reactive, onMounted, computed} from "vue"
import socketManager from '../socketManager.js'
import TexRenderer from './TexRenderer.vue'

// #region global state
const userName = inject("userName")
// #endregion

// #region local variable
const socket = socketManager.getInstance()
// #endregion

// Preview-Tex
const previewTex = computed(() => {
  const parts = chatContent.value.split('tex:');
  return parts.length > 1 ? parts[1].trim() : null; // LaTeX部分
})

// Preview-Text
const previewText = computed(() => {
  const parts = chatContent.value.split('tex:');
  return parts[0].trim(); // 通常テキスト部分
})

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

// 投稿メッセージをサーバに送信
const onPublish = () => {
  const message = `${userName.value}さん: ${chatContent.value}`
  socket.emit("publishEvent", message)
  // 入力欄を初期化
  chatContent.value = ""
}

// 退室メッセージをサーバに送信する
const onExit = () => {
  const message = `${userName.value}さんが退室しました。`
  socket.emit("exitEvent", message)
}

// メモを取る
const onMemo = () => {
  const message = `${userName.value}さんのメモ: ${chatContent.value}`
  processMessage(message)
  // 入力欄を初期化
  chatContent.value = ""
}
// #endregion

// #region socket event handler
const onReceiveEnter = (data) => {
  const loginMessage = `${data}さんが入室しました`
  chatList.unshift({
    text: loginMessage,
    isTex: false
  })
}

const onReceiveExit = (data) => {
  chatList.unshift({
    text: data,
    isTex: false
  })
}

const onReceivePublish = (data) => {
  processMessage(data);
  
}
// #endregion

// メッセージを処理するヘルパー関数
const processMessage = (message) => {
  const parts = message.split('tex:');
  const normalText = parts[0].trim();
  const texText = parts.slice(1).join('tex:').trim();


  // 分割後の各部分をログに表示
  console.log("Message parts:", parts);
  
  if (texText) {
    chatList.unshift({
      text: texText,
      isTex: true
    });
    console.log("Normal text:", normalText);
  }

  if (normalText) {
    chatList.unshift({
      text: normalText,
      isTex: false
    });
    console.log("LaTeX text:", texText);
  }
}

// #region local methods
const registerSocketEvent = () => {
  socket.on("enterEvent", (data) => {
    onReceiveEnter(data)
  })

  socket.on("exitEvent", (data) => {
    onReceiveExit(data)
  })

  socket.on("publishEvent", (data) => {
    onReceivePublish(data)
  })
}

// メッセージがメモかどうかを判定する
const isMemo = (chat) => {
  return chat.isMemo
}

//Shift Enterの改行を実行するメソッド
const KeyEnterShift = () => {
  return 
}

// #endregion
</script>

<template>
  <div class="mx-auto my-5 px-4">
    <h1 class="text-h3 font-weight-medium">Vue.js Chat チャットルーム</h1>
    <div class="mt-10">
      <p>ログインユーザ：{{ userName }}さん</p>
      <textarea v-model="chatContent" @keydown.enter.exact="onPublish" @keydown.enter.shift.exact="KeyEnterShift" variant="outlined" placeholder="投稿文を入力してください" rows="4" class="area"></textarea>
      <div class="mt-5">
        <button class="button-normal" @click="onPublish">投稿</button>
        <button class="button-normal util-ml-8px" @click="onMemo">メモ</button>
      </div>

      <!-- Preview Area -->
      <div class="preview mt-5">
        <h3>Preview</h3>
        <!-- Text & Tex -->
        <span v-if="!previewTex">{{ chatContent }}</span>
        <span v-else>
          <!-- texの前のText部分を表示 -->
          {{ previewText }}
          <!-- tex部分を表示 -->
          <TexRenderer v-if="previewTex" :formula="previewTex" />
        </span>
      </div>

      <div class="mt-5" v-if="chatList.length !== 0">
        <ul>
          <li class="item mt-4" v-for="(chat, i) in chatList" :key="i" :class="{ 
            publish: chat.text.startsWith(userName + 'さん:'), 
            memo: chat.text.startsWith(userName + 'さんのメモ:'), 
            tex: chat.isTex
          }">
            <span v-if="!chat.isTex">{{ chat.text }}</span>
            <TexRenderer v-if="chat.isTex" :formula="chat.text" :key="chat.text" />
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

/* Preview-Style */
.preview {
  background-color: #f0f0f0;
  padding: 10px;
  border: 1px solid #ddd;
}

.publish {
  color: red !important;
  font-style: italic !important;
}

.memo {
  color: blue;
  /* 青色のテキスト */
}

.tex {
  color: green; /* 緑色のテキスト */
  background-color: #f8f9fa !important;
  padding: 5px !important;
  border-left: 4px solid #ced4da !important;
  margin-top: 10px !important;
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
