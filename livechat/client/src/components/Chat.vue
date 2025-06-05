<template>
  <div class="chat-app font-sans bg-gray-100 min-h-screen flex flex-col items-center justify-center p-4">
    <div class="chat-container w-full max-w-2xl bg-white rounded-lg shadow-xl p-6">
      <h1 class="text-3xl font-bold text-center text-indigo-600 mb-6">Vue 3 WebSocket Chat</h1>

      <div
        class="messages-area bg-gray-50 p-4 overflow-y-auto rounded-md border border-gray-300 mb-4"
        ref="messagesArea"
      >
        <div v-if="messages.length === 0" class="text-gray-500 text-center py-4">
          No messages yet. Start chatting!
        </div>
        <div v-for="(msg, index) in messages" :key="index"
             class="message p-3 my-2 rounded-lg shadow-sm break-words"
             :class="msg.type === 'sent' ? 'bg-indigo-500 text-white ml-auto self-sent' :
                      msg.type === 'received' ? 'bg-gray-200 text-gray-800 mr-auto self-received' :
                      'bg-yellow-100 text-yellow-700 text-center self-info'">
          <span v-if="msg.type === 'sent'" class="font-semibold">You: </span>
          <span v-if="msg.type === 'received'" class="font-semibold">Other: </span>
          {{ msg.text }}
        </div>
      </div>

      <form @submit.prevent="sendMessage" class="message-input-area flex gap-3">
        <input
          v-model="newMessage"
          placeholder="Type your message..."
          class="flex-grow p-4 border border-gray-300 rounded-md focus:ring-2 focus:ring-indigo-500 focus:border-transparent outline-none transition-shadow text-lg"
          :disabled="!isConnected"
          style="height: 3.5rem;"
        />
        <button
          type="submit"
          class="bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-3 px-6 rounded-md shadow-md hover:shadow-lg transition-all duration-150 ease-in-out disabled:opacity-50 disabled:cursor-not-allowed"
          :disabled="!newMessage.trim() || !isConnected"
        >
          Send
        </button>
      </form>

      <div v-if="!isConnected && !isConnecting" class="mt-4 text-center">
        <button
          @click="connectWebSocket"
          class="bg-blue-500 hover:bg-blue-600 text-white font-semibold py-2 px-4 rounded-md shadow-md hover:shadow-lg transition-all duration-150 ease-in-out"
        >
          Reconnect
        </button>
      </div>
    </div>
     <footer class="text-center text-sm text-gray-500 mt-10">
        Open this page in two browser windows to chat. <br/>
        Ensure the WebSocket server (server.js) is running on ws://localhost:8080.
    </footer>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  ref,
  onMounted,
  onUnmounted,
  computed,
  watch,
  nextTick
} from 'vue';

interface Message {
  text: string;
  type: 'sent' | 'received' | 'info';
}

export default defineComponent({
  name: 'Chat',
  setup() {
    const messages = ref<Message[]>([]);
    const newMessage = ref<string>('');
    const socket = ref<WebSocket | null>(null);
    const isConnected = ref<boolean>(false);
    const isConnecting = ref<boolean>(false);
    const messagesArea = ref<HTMLElement | null>(null);

    const connectionStatus = computed(() => {
      if (isConnecting.value) return 'Connecting...';
      return isConnected.value ? 'Connected to chat server' : 'Disconnected from chat server';
    });

    const connectWebSocket = () => {
      if (socket.value && socket.value.readyState === WebSocket.OPEN) {
        console.log('Already connected.');
        return;
      }

      console.log('Attempting to connect to WebSocket server...');
      isConnecting.value = true;
      isConnected.value = false;

      const wsUrl = 'ws://localhost:8080';
      socket.value = new WebSocket(wsUrl);

      socket.value.onopen = () => {
        console.log('WebSocket connection established.');
        isConnected.value = true;
        isConnecting.value = false;
        messages.value.push({ text: 'Connected to the server!', type: 'info' });
      };

      socket.value.onmessage = (event) => {
        console.log('Message from server: ', event.data);
        try {
            const receivedData = JSON.parse(event.data as string);
            if (receivedData.type === 'info') {
                 messages.value.push({ text: receivedData.message, type: 'info' });
            } else {
                 messages.value.push({ text: receivedData as unknown as string, type: 'received' });
            }
        } catch (error) {
            messages.value.push({ text: event.data as string, type: 'received' });
        }
      };

      socket.value.onclose = (event) => {
        console.log('WebSocket connection closed.', event);
        isConnected.value = false;
        isConnecting.value = false;
        messages.value.push({ text: 'Disconnected from server.', type: 'info' });
      };

      socket.value.onerror = (error) => {
        console.error('WebSocket error: ', error);
        isConnected.value = false;
        isConnecting.value = false;
        messages.value.push({ text: 'Connection error.', type: 'info' });
      };
    };

    const sendMessage = () => {
      if (newMessage.value.trim() && socket.value && socket.value.readyState === WebSocket.OPEN) {
        const messageToSend = newMessage.value.trim();
        socket.value.send(messageToSend);
        messages.value.push({ text: messageToSend, type: 'sent' });
        newMessage.value = '';
      } else {
        console.log('Cannot send message. Socket not connected or message is empty.');
         if (!socket.value || socket.value.readyState !== WebSocket.OPEN) {
            messages.value.push({ text: 'Not connected. Cannot send message.', type: 'info' });
        }
      }
    };

    const scrollToBottom = () => {
      if (messagesArea.value) {
        messagesArea.value.scrollTop = messagesArea.value.scrollHeight;
      }
    };

    onMounted(() => {
      connectWebSocket();
    });

    onUnmounted(() => {
      if (socket.value) {
        socket.value.close();
        console.log('WebSocket connection closed on component unmount.');
      }
    });

    watch(
      () => messages.value.length,
      async () => {
        await nextTick();
        scrollToBottom();
      }
    );

    return {
      messages,
      newMessage,
      sendMessage,
      isConnected,
      isConnecting,
      connectionStatus,
      connectWebSocket,
      messagesArea,
    };
  },
});
</script>

<style scoped>

.chat-app {
  font-family: 'Inter', sans-serif;
}

.messages-area {
  display: flex;
  flex-direction: column;
  height: 24rem;
  max-height: 35vh;
  overflow-y: auto;
  padding-right: 0.5rem;
  box-sizing: border-box;
}

.message-input-area input {
  max-height: 1rem;
  font-size: 1.2rem;
  padding: 1rem;
}

.message {
  max-width: 65%;
  word-wrap: break-word;
}

.self-sent {
  align-self: flex-end;
}

.self-received {
  align-self: flex-start;
}

.self-info {
  align-self: center;
  font-style: italic;
  font-size: 0.9em;
}

.messages-area::-webkit-scrollbar {
  width: 8px;
  background: transparent;
}

.messages-area::-webkit-scrollbar-track {
  background: transparent;
}

.messages-area::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 10px;
  border: 3px solid #f9fafb;
}

.messages-area::-webkit-scrollbar-thumb:hover {
  background: #a0aec0;
}
</style>
