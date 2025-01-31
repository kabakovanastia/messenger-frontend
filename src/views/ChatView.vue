<script setup lang="ts"></script>

<template>
  <div class="container">
    <div class="info-container">
      <span class="info-title">Username</span>
      <input type="text" class="info-inp" />
      <button class="infobtn">Set username</button>

      <span class="info-title top-border">Chat with:</span>
      <input type="text" class="info-inp" />
      <button class="infobtn">Chat</button>
    </div>
    <div class="chat">
      <span class="friend-name">Nania</span>
      <div class="messages">
        <span class="message message-from">Hey lalalla</span>
        <span class="message message-to">Hello</span>
        <span class="message message-to">MImimimimim asda dsa sadsad sad a</span>
        <span class="message message-to"
          >dsaf sdfn ksjndf sdkf sdlfn saf asnfl sdfsdf asf safsdf sdf saf
        </span>
        <span class="message message-from">Hey lalalla</span>
        <span class="message message-to">Hello</span>
        <span class="message message-to">MImimimimim asda dsa sadsad sad a</span>
        <span class="message message-to"
          >dsaf sdfn ksjndf sdkf sdlfn saf asnfl sdfsdf asf safsdf sdf saf
        </span>
        <span class="message message-from">Hey lalalla</span>
        <span class="message message-to">Hello</span>
        <span class="message message-from">MImimimimim asda dsa sadsad sad a</span>
        <span class="message message-to"
          >dsaf sdfn ksjndf sdkf sdlfn saf asnfl sdfsdf asf safsdf sdf saf
        </span>
        <span class="message message-from">Hey lalalla</span>
        <span class="message message-from">Hello</span>
        <span class="message message-to">MImimimimim asda dsa sadsad sad a</span>
        <span class="message message-to"
          >dsaf sdfn ksjndf sdkf sdlfn saf asnfl sdfsdf asf safsdf sdf saf
        </span>
        <span class="message message-from">fd fsdf asf sdfafas fsdf sdf sdfsdadf sd fsadfas f</span>
      </div>
      <div class="write-line">
        <input class="inp-text" type="text" />
        <button class="sendbtn">Send</button>
      </div>
    </div>
  </div>
</template>

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
  height: 50px;
  font-size: 25px;
  border-bottom: 2px solid black;
}

.write-line {
  display: flex;
  flex-direction: row;
  padding: 5px 15px;
  padding-bottom: 10px;

  height: 50px;

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
