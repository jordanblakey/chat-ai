<script setup lang="ts">
import { onMounted, nextTick } from "vue";
import { useUserStore } from "../stores/user";
import { useChatStore } from "../stores/chat";
import { useRouter } from "vue-router";
import Header from "../components/Header.vue";
import ChatInput from "../components/ChatInput.vue";

const userStore = useUserStore();
const chatStore = useChatStore();
const router = useRouter();

// Ensure user is logged in
if (!userStore.userId) {
  router.push("/");
}

// Format AI messages for better display
const formatMessage = (text: string) => {
  if (!text) return "";

  return text
    .replace(/\n/g, "<br>")
    .replace(/\*\*(.*?)\*\*/g, "<b>$1</b>")
    .replace(/\*(.*?)\*/g, "<i>$1</i>")
    .replace(/`(.*)`/g, "<code>$1</code>")
    .replace(/(?:^|\n)- (.*?)(?:\n|$)/g, "<li>$1</li>")
    .replace(/(?:^|\n)(\d+)\. (.*?)(?:\n|$)/g, "<li>$1. $2</li>")
    .replace(/<\/li>\n<li>/g, "</li><li>")
    .replace(/<li>/, "<ul><li>")
    .replace(/<\/li>$/, "</li></ul>");
};

// Auto-scroll to bottom
const scrollToBottom = () => {
  nextTick(() => {
    const chatContainer = document.getElementById("chat-container");
    if (chatContainer) chatContainer.scrollTop = chatContainer.scrollHeight;
  });
};

onMounted(() => {
  chatStore.loadChatHistory().then(() => scrollToBottom());
  console.log("chatStore.messages", chatStore.messages);
});
</script>

<template>
  <div class="flex flex-col h-screen bg-gray-900 text white">
    <Header></Header>

    <!-- Chat messages -->
    <div id="chat-container" class="flex-1 overflow-y-auto p-4 space-y-4">
      <div
        class="flex items-start"
        v-for="(msg, index) in chatStore.messages"
        :key="index"
        :class="msg.role === 'user' ? 'justify-end' : 'justify-start'"
      >
        <div
          v-html="formatMessage(msg.content)"
          class="max-w-xs px-4 py-2 rounded-lg md:max-w-md"
          :class="
            msg.role === 'user'
              ? 'bg-blue-600 text-white'
              : 'bg-gray-700 text-white'
          "
        ></div>
      </div>
      <div v-if="chatStore.isLoading" class="flex-justify-start">
        <div class="bg-gray-700 text-white px-4 py-2 rounded-lg">
          <span class="animate-pulse">AI is thinking...</span>
        </div>
      </div>
    </div>
    <ChatInput @send="chatStore.sendMessage"></ChatInput>
  </div>
</template>
