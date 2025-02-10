<template>
  <div class="container">
    <div class="info-container">
      <span ref="username_span" class="info-title">{{ username ? username : 'Username' }}</span>
      <input ref="username_inp" type="text" class="info-inp" />
      <button @click="set_username" class="infobtn">Set username</button>

      <span class="info-title top-border">Chat with:</span>
      <input ref="friend_inp" type="text" class="info-inp" />
      <button @click="set_friend" class="infobtn">Set chat name</button>
      <!-- <button @click="get_messages_since">update</button> -->
      <!-- <button @click="sock_conn">sock conn</button> -->
    </div>
    <div class="chat">
      <span ref="friend_span" class="friend-name">{{ chat_name ? chat_name : 'Chat name' }}</span>
      <div ref="messages" class="messages">
        <span
          class="message"
          v-for="msgitem in data"
          :class="msgitem.msg_from == un_span.textContent ? 'message-to' : 'message-from'"
          :key="msgitem.id"
        >
          {{ msgitem.text }}
        </span>
      </div>
      <div class="write-line">
        <input ref="message_inp" class="inp-text" type="text" />
        <button ref="send_btn" @click="send_message" class="sendbtn">Send</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import axios from 'axios'
import { onUpdated, ref, useTemplateRef } from 'vue'

const data = ref([])
const username = ref('')
const chat_name = ref('')
const un_inp = useTemplateRef('username_inp')
const un_span = useTemplateRef('username_span')
const fr_inp = useTemplateRef('friend_inp')
const fr_span = useTemplateRef('friend_span')
const msg_inp = useTemplateRef('message_inp')
const msgs = useTemplateRef('messages')

const DB_VERSION = 4

let web_sock
const openRequest = indexedDB.open('messages', DB_VERSION)
openRequest.onupgradeneeded = function (event) {
  const db = event.target.result
  console.log(db.objectStoreNames)
  if (!db.objectStoreNames.contains('msgs')) {
    db.createObjectStore('msgs', { keyPath: 'id' })
  }
}

onUpdated(() => {
  msgs.value.scrollTop = msgs.value.scrollHeight
})

async function get_messages() {
  const username_from = username.value.trim()
  const username_to = chat_name.value.trim()
  if (!username_from || !username_to) return

  const openRequest = indexedDB.open('messages', DB_VERSION)
  openRequest.onsuccess = function () {
    const db = openRequest.result

    const temp_data = []
    const msgs_trans = db.transaction('msgs', 'readonly').objectStore('msgs')
    const cursorRequest = msgs_trans.openCursor()
    cursorRequest.onsuccess = async function () {
      const cursor = cursorRequest.result
      if (cursor) {
        const cur_msg = cursor.value
        if (
          (cur_msg['msg_from'] == username_from && cur_msg['msg_to'] == username_to) ||
          (cur_msg['msg_from'] == username_to && cur_msg['msg_to'] == username_from)
        ) {
          temp_data.push(cur_msg)
        }
        cursor.continue()
      } else {
        if (temp_data.length == 0) {
          axios
            .get(`http://127.0.0.1:8000/api/all/?p1=${username_from}&p2=${username_to}`)
            .then((response) => {
              const messages = response.data
              data.value = messages
              const msgs_trans = db.transaction('msgs', 'readwrite').objectStore('msgs')
              for (const item of messages) {
                msgs_trans.add(item)
              }
            })
            .catch(function (error) {
              // console.log(error)
            })
        } else {
          data.value = temp_data
          get_messages_since()
        }
      }
    }
  }
}

async function get_messages_since() {
  if (data.value.length == 0) {
    get_messages()
    return
  }

  const username_from = username.value.trim()
  const username_to = chat_name.value.trim()
  const last_msg_datetime = data.value[data.value.length - 1]['datetime']

  axios
    .get(
      `http://127.0.0.1:8000/api/all/?since=${last_msg_datetime}&p1=${username_from}&p2=${username_to}`,
    )
    .then((response) => {
      const messages = response.data
      data.value.push(...messages)

      const openRequest = indexedDB.open('messages', DB_VERSION)
      openRequest.onsuccess = function () {
        const db = openRequest.result
        const msgs_trans = db.transaction('msgs', 'readwrite').objectStore('msgs')
        for (const item of messages) {
          msgs_trans.add(item)
        }
      }
    })
    .catch(function (error) {
      // console.log(error)
    })
}

async function sock_conn() {
  const username_from = username.value.trim()
  const username_to = chat_name.value.trim()

  if (!username_from || !username_to) return

  const chat_name_string = get_chat_name_string(username_from, username_to)

  if (web_sock) {
    web_sock.close()
  }

  web_sock = new WebSocket(`http://127.0.0.1:8001/ws/chat/${chat_name_string}/`)

  web_sock.onmessage = () => {
    console.log('get message')
    get_messages_since()
  }

  web_sock.onopen = () => {
    console.log('connection opened')
  }
}

async function send_message() {
  const username_from = un_span.value.textContent.trim()
  const username_to = fr_span.value.textContent.trim()
  const text = msg_inp.value?.value
  if (!text) return

  axios
    .post('http://127.0.0.1:8000/api/create/', {
      msg_from: username_from,
      msg_to: username_to,
      text: text,
    })
    .then((response) => {
      const message = response.data
      data.value.push(message)

      const openRequest = indexedDB.open('messages', DB_VERSION)
      openRequest.onsuccess = function () {
        const db = openRequest.result
        console.log()
        const msgs_trans = db.transaction('msgs', 'readwrite').objectStore('msgs')
        msgs_trans.add(message)

        if (web_sock) {
          web_sock.send('sent message')
        }

        msg_inp.value.value = ''
        msg_inp.value.focus()
      }
    })
    .catch(function (error) {
      // console.log(error)
    })
}

function set_username() {
  if (un_inp.value?.value.trim()) {
    username.value = un_inp.value?.value.trim()
    get_messages()
    sock_conn()
  }
}

function set_friend() {
  if (fr_inp.value?.value.trim()) {
    chat_name.value = fr_inp.value?.value.trim()
    get_messages()
    sock_conn()
  }
}

function get_chat_name_string(p1, p2) {
  return (p1 > p2 ? p1 + p2 : p2 + p1).toString()
}

document.addEventListener('keydown', function (event) {
  if (event.code == 'Enter') {
    send_message()
  }
})
</script>

<style scoped>
.top-border {
  border-top: 2px solid black;
  width: 100%;
  text-align: center;
  padding-top: 10px;
}
.info-title {
  margin-top: 10px;
  font-size: 22px;
}
.info-inp {
  background-color: inherit;
  border: none;
  width: 80%;
  border-radius: 3px;
  border: 2px solid black;
  margin-top: 10px;

  font-size: 22px;
}
.info-inp:focus {
  outline: none;
}
.infobtn {
  background-color: inherit;
  border-radius: 1000px;
  border: 5px solid black;
  font-size: 20px;
  padding: 5px;

  margin-top: 10px;
  margin-bottom: 10px;
}
.infobtn:hover {
  cursor: pointer;
}
.infobtn:active {
  background-color: rgb(158, 158, 158);
}

.container {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;

  margin-top: 25px;

  width: 100%;
  height: 90vh;
}

.friend-name {
  text-align: center;
  min-height: 50px;
  font-size: 25px;
  border-bottom: 2px solid black;
}

.write-line {
  display: flex;
  flex-direction: row;
  padding: 5px 15px;
  padding-bottom: 10px;

  min-height: 50px;

  border-top: 2px solid black;
}

.inp-text {
  flex-grow: 1;
  height: 100%;
  background-color: inherit;
  border: none;

  font-size: 22px;
}

.inp-text:focus {
  outline: none;
}

.sendbtn {
  /* width: 60px; */
  height: 100%;

  margin-left: 20px;

  background-color: inherit;
  border-radius: 1000px;
  border: 5px solid black;
  font-size: 20px;
  padding: 5px;
}

.sendbtn:hover {
  cursor: pointer;
}

.sendbtn:active {
  background-color: rgb(158, 158, 158);
}

.info-container {
  display: flex;
  flex-direction: column;

  align-items: center;

  margin-right: 40px;

  width: 20vw;
  height: 100%;
  max-width: 280px;

  border-radius: 20px;
  border: 5px solid black;
}

.chat {
  display: flex;
  flex-direction: column;

  width: 60vw;
  height: 100%;
  max-width: 750px;

  border-radius: 20px;
  border: 5px solid black;
}

.myname {
  font-size: 20px;
  color: black;
  margin-bottom: 10px;
  padding: 15px;
  font-weight: 600;
}

.friend-name {
  color: black;
}

.messages {
  display: flex;
  flex-direction: column;

  flex-grow: 1;

  overflow-y: scroll;
}

.message {
  padding: 5px;
  margin: 3px;
  border-radius: 5px;
  color: black;
  max-width: 300px;

  margin: 4px 15px;
}

.message-from {
  background-color: rgb(101, 205, 101);
  align-self: flex-start;
}

.message-to {
  background-color: rgb(214, 194, 105);
  align-self: flex-end;
}
</style>
