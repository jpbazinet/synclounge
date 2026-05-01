<template>
  <v-list-item>
    <v-list-item-avatar size="32">
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
.message-content {
  white-space: normal !important;
  font-weight: normal !important;

  /* Allow long URLs to wrap instead of overflowing */
  word-break: break-word;
  overflow-wrap: anywhere;
}

/* stylelint-disable-next-line selector-pseudo-class-no-unknown */
.message-content :deep(.chat-link) {
  color: inherit;
  text-decoration: underline;
  word-break: break-all;
}

/* stylelint-disable-next-line selector-pseudo-class-no-unknown */
.message-content :deep(.chat-link:hover) {
  opacity: 0.8;
}
</style>
