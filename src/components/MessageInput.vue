<template>
  <div class="message-input-wrapper">
    <v-menu
      v-model="emojiPickerOpen"
      :close-on-content-click="false"
      top
      offset-y
      nudge-top="8"
      max-width="320"
    >
      <template #activator="{ on, attrs }">
        <v-btn
          icon
          small
          class="emoji-btn ml-1"
          v-bind="attrs"
          title="Emoji"
          v-on="on"
        >
          <v-icon small>
            mdi-emoticon-outline
          </v-icon>
        </v-btn>
      </template>
      <v-card class="emoji-picker pa-2">
        <div class="emoji-search mb-1">
          <v-text-field
            v-model="emojiSearch"
            dense
            hide-details
            placeholder="Search emoji…"
            prepend-inner-icon="mdi-magnify"
            outlined
            @click.stop
          />
        </div>
        <div class="emoji-grid">
          <button
            v-for="e in filteredEmojis"
            :key="e.emoji"
            class="emoji-item"
            :title="e.name"
            @click.stop="insertEmoji(e.emoji)"
          >
            {{ e.emoji }}
          </button>
        </div>
      </v-card>
    </v-menu>

    <v-text-field
      ref="messageInput"
      v-model="messageToBeSent"
      append-outer-icon="mdi-send"
      label="Message"
      hide-details
      single-line
      class="ml-1 mr-2 pr-1"
      @click:append-outer="sendMessage"
      @keyup.enter.native="sendMessage"
    />
  </div>
</template>

<script>
import { mapActions } from 'vuex';

const EMOJIS = [
  { emoji: '😀', name: 'grinning' },
  { emoji: '😂', name: 'joy' },
  { emoji: '🤣', name: 'rofl' },
  { emoji: '😍', name: 'heart eyes' },
  { emoji: '🥰', name: 'smiling face with hearts' },
  { emoji: '😎', name: 'cool' },
  { emoji: '🤔', name: 'thinking' },
  { emoji: '😭', name: 'crying' },
  { emoji: '😱', name: 'scream' },
  { emoji: '🤯', name: 'exploding head' },
  { emoji: '😤', name: 'triumph' },
  { emoji: '🥳', name: 'partying' },
  { emoji: '😴', name: 'sleeping' },
  { emoji: '🤮', name: 'vomiting' },
  { emoji: '😬', name: 'grimacing' },
  { emoji: '🙄', name: 'eye roll' },
  { emoji: '😏', name: 'smirk' },
  { emoji: '🤡', name: 'clown' },
  { emoji: '👍', name: 'thumbs up' },
  { emoji: '👎', name: 'thumbs down' },
  { emoji: '👏', name: 'clap' },
  { emoji: '🙌', name: 'raised hands' },
  { emoji: '🤝', name: 'handshake' },
  { emoji: '✌️', name: 'peace' },
  { emoji: '🤞', name: 'fingers crossed' },
  { emoji: '💪', name: 'muscle' },
  { emoji: '🫶', name: 'heart hands' },
  { emoji: '❤️', name: 'heart' },
  { emoji: '🔥', name: 'fire' },
  { emoji: '💀', name: 'skull' },
  { emoji: '💩', name: 'poop' },
  { emoji: '👀', name: 'eyes' },
  { emoji: '🎉', name: 'party' },
  { emoji: '🍿', name: 'popcorn' },
  { emoji: '🎬', name: 'clapper' },
  { emoji: '📽️', name: 'film projector' },
  { emoji: '🍕', name: 'pizza' },
  { emoji: '🍺', name: 'beer' },
  { emoji: '☕', name: 'coffee' },
  { emoji: '🤦', name: 'facepalm' },
  { emoji: '🤷', name: 'shrug' },
  { emoji: '💯', name: '100' },
  { emoji: '✨', name: 'sparkles' },
  { emoji: '😆', name: 'laughing' },
  { emoji: '😅', name: 'sweat smile' },
  { emoji: '🫠', name: 'melting' },
  { emoji: '🥱', name: 'yawning' },
  { emoji: '😐', name: 'neutral' },
];

export default {
  name: 'MessageInput',

  data: () => ({
    messageToBeSent: '',
    emojiPickerOpen: false,
    emojiSearch: '',
  }),

  computed: {
    filteredEmojis() {
      if (!this.emojiSearch) return EMOJIS;
      const q = this.emojiSearch.toLowerCase();
      return EMOJIS.filter((e) => e.name.includes(q));
    },
  },

  methods: {
    ...mapActions('synclounge', [
      'SEND_MESSAGE',
    ]),

    sendMessage() {
      if (this.messageToBeSent.trim() === '') return;
      this.SEND_MESSAGE(this.messageToBeSent);
      this.messageToBeSent = '';
    },

    insertEmoji(emoji) {
      const input = this.$refs.messageInput.$el.querySelector('input');
      if (!input) {
        this.messageToBeSent += emoji;
        return;
      }
      const start = input.selectionStart ?? this.messageToBeSent.length;
      const end = input.selectionEnd ?? start;
      this.messageToBeSent = this.messageToBeSent.slice(0, start) + emoji + this.messageToBeSent.slice(end);
      this.$nextTick(() => {
        input.focus();
        const pos = start + emoji.length;
        input.setSelectionRange(pos, pos);
      });
    },
  },
};
</script>

<style scoped>
.message-input-wrapper {
  display: flex;
  align-items: center;
  width: 100%;
}

.emoji-btn {
  flex-shrink: 0;
}

.emoji-picker {
  width: 300px;
}

.emoji-grid {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 2px;
  max-height: 200px;
  overflow-y: auto;
}

.emoji-item {
  font-size: 20px;
  padding: 4px;
  border: none;
  background: transparent;
  cursor: pointer;
  border-radius: 4px;
  line-height: 1;
  transition: background 0.15s;
}

.emoji-item:hover {
  background: rgba(128, 128, 128, 0.2);
}
</style>
