<script setup>
import { io } from "socket.io-client";
import { onBeforeMount, ref } from "vue";

// aderir ao socket que rota no 3001
const socket = io("http://localhost:3001");

// incia de constantes e tipos
const messages = ref([]);
const messageText = ref("");
const joined = ref(false);
const name = ref('');
const typingDisplay = ref('');

// método
onBeforeMount(() => {
  // nome do método - payload - response em callback
  socket.emit("findAllMessages", {}, (response) => {
    // response -> mensagens que retornamos do servidor
    messages.value = response;
  });

  // pega mensagem e insere em array
  socket.on("message", (message) => {
    messages.value.push(message);
  });
});

// método
const join = () = {
  socket.emit('join', { name: name.value}, () => {
    joined.value = true;
  })
}

// método
const sendMessage = () => {
  // nome do método -> payload -> response em callback
  socket.emit("createMessage", { text: messageText.value }, (response) => {
    // response -> apagar input de mensagem
    messageText.value = "";
  });
};

// método
let timeout;
const emitTyping = () => {
  // emitir evento typing
  socket.emit('typing', { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit('typing', {isTyping: false});
  }, 2000);
}
</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label>What's your name?</label>
        <input v-model="name" />
        <button type="submit">Send</button>
      </form>
    </div>
    <div class="chat-container" v-else>
      <div class="messages-container">
        <div v-for="message in messages">
          [{{ message.name }}]: {{ message.text }}
        </div>
      </div>

      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messageText" @input="emitTyping" />
          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style>
@import "./assets/base.css";
</style>
