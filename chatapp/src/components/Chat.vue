<script setup>
import { inject, ref, reactive, onMounted } from "vue"
import socketManager from '../socketManager.js'
import TexRenderer from './TexRenderer.vue'

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
const onPublish = () => {
  const message = `${userName.value}さん: ${chatContent.value}`
  socket.emit("publishEvent", message)

  // 入力欄を初期化
  chatContent.value = ""
}

const onExit = () => {}

const onMemo = () => {
  const message = `${userName.value}さんのメモ: ${chatContent.value}`
  
  // メッセージ内に `tex:` プレフィックスが含まれている場合、その後の部分を LaTeX として処理
  const parts = message.split('tex:');
  
  if (parts.length > 1) {
    const normalText = parts[0].trim();
    const texText = parts.slice(1).join('tex:').trim();
    
    // LaTeX 部分を追加
    if (texText) {
      chatList.unshift({
        text: texText, // LaTeX 部分の前に半角スペースを追加
        isTex: true
      });
    }

    // メッセージの通常部分を追加
    if (normalText) {
      chatList.unshift({
        text: normalText,
        isTex: false
      });
    }

    
  } else {
    // `tex:` プレフィックスが含まれていない場合、そのまま表示
    chatList.unshift({
      text: message,
      isTex: false
    });
  }

  // 入力欄を初期化
  chatContent.value = ""
}
// #endregion

// #region socket event handler
const onReceiveEnter = (data) => {
  chatList.unshift({
    text: data,
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
  // メッセージ内に `tex:` プレフィックスが含まれている場合、その後の部分を LaTeX として処理
  const parts = data.split('tex:');
  
  if (parts.length > 1) {
    const normalText = parts[0].trim();
    const texText = parts.slice(1).join('tex:').trim();
    
    // LaTeX 部分を追加
    if (texText) {
      chatList.unshift({
        // text: `\\text{　Tex: } ${texText}`,
        text: texText,
        isTex: true
      });
    }

    // メッセージの通常部分を追加
    if (normalText) {
      chatList.unshift({
        text: normalText,
        isTex: false
      });
    }

    
  } else {
    // `tex:` プレフィックスが含まれていない場合、そのまま表示
    chatList.unshift({
      text: data,
      isTex: false
    });
  }
}
// #endregion

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
          <li class="item mt-4" v-for="(chat, i) in chatList" :key="i" :class="{ 
            publish: chat.text.startsWith(userName + 'さん:'), 
            memo: chat.text.startsWith(userName + 'さんのメモ:'), 
            tex: chat.isTex
          }">
            <span v-if="!chat.isTex">{{ chat.text }}</span>
            <TexRenderer v-if="chat.isTex" :formula="chat.text" />
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
}

.memo {
  color: blue; /* 青色のテキスト */
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
