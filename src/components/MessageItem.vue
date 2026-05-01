<template>
  <v-list-item class="message-item">
    <v-list-item-avatar size="40">
      <v-img :src="sender.thumb" />
    </v-list-item-avatar>
    <v-list-item-content>
      <v-list-item-title v-text="sender.username" />
      <!-- eslint-disable-next-line vue/no-v-html -->
      <v-list-item-subtitle class="message-content" v-html="processedText" />
    </v-list-item-content>
  </v-list-item>
</template>

<script>
import { mapGetters } from 'vuex';

// Matches http/https URLs
const URL_PATTERN = /(https?:\/\/[^\s<>"']+)/g;

function escapeHtml(text) {
  return text
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
}

export default {
  name: 'MessageItem',
  props: {
    message: {
      type: Object,
      required: true,
    },
  },
  computed: {
    ...mapGetters('synclounge', [
      'GET_MESSAGES_USER_CACHE_USER',
    ]),
    sender() {
      return this.GET_MESSAGES_USER_CACHE_USER(this.message.senderId);
    },
    processedText() {
      if (this.message.text.startsWith('data:image/')) {
        const src = this.message.text;
        const imgStyle = 'max-width:100%;max-height:300px;border-radius:4px;display:block;';
        return `<img src="${src}" style="${imgStyle}" />`;
      }
      const safe = escapeHtml(this.message.text);
      return safe.replace(URL_PATTERN, (url) => {
        const attrs = `href="${url}" target="_blank" rel="noopener noreferrer" class="chat-link"`;
        return `<a ${attrs}>${url}</a>`;
      });
    },
  },
};
</script>

<style scoped>
.message-item {
  align-items: flex-start !important;
  padding: 10px 12px !important;
}
.message-item + .message-item {
  border-top: 1px solid rgba(255, 255, 255, 0.05);
}
.message-item :deep(.v-list-item__avatar) {
  margin-top: 2px;
  margin-right: 10px;
}
.message-item :deep(.v-avatar) {
  box-shadow: 0 0 0 2px rgba(229, 160, 13, 0.5) !important;
}
.message-item :deep(.v-list-item__title) {
  color: #f0a020 !important;
  font-weight: 600 !important;
  font-size: 13px !important;
  letter-spacing: 0.01em !important;
  margin-bottom: 3px !important;
}
.message-content {
  white-space: normal !important;
  font-weight: normal !important;
  word-break: break-word;
  overflow-wrap: anywhere;
  color: rgba(255, 255, 255, 0.85) !important;
  line-height: 1.5 !important;
}
.message-content :deep(.chat-link) {
  color: inherit;
  text-decoration: underline;
  word-break: break-all;
}
.message-content :deep(.chat-link:hover) {
  opacity: 0.8;
}
</style>
