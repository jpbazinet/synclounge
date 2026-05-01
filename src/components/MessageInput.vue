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
            placeholder="Search emojiâ¦"
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
      @keyup.space.native="convertEmoticons"
    />
  </div>
</template>

<script>
import { mapActions } from 'vuex';

const EMOJIS = [
  { emoji: 'ð', name: 'grinning' },
  { emoji: 'ð', name: 'joy' },
  { emoji: 'ð¤£', name: 'rofl' },
  { emoji: 'ð', name: 'heart eyes' },
  { emoji: 'ð¥°', name: 'smiling face with hearts' },
  { emoji: 'ð', name: 'cool' },
  { emoji: 'ð¤', name: 'thinking' },
  { emoji: 'ð­', name: 'crying' },
  { emoji: 'ð±', name: 'scream' },
  { emoji: 'ð¤¯', name: 'exploding head' },
  { emoji: 'ð¤', name: 'triumph' },
  { emoji: 'ð¥³', name: 'partying' },
  { emoji: 'ð´', name: 'sleeping' },
  { emoji: 'ð¤®', name: 'vomiting' },
  { emoji: 'ð¬', name: 'grimacing' },
  { emoji: 'ð', name: 'eye roll' },
  { emoji: 'ð', name: 'smirk' },
  { emoji: 'ð¤¡', name: 'clown' },
  { emoji: 'ð', name: 'thumbs up' },
  { emoji: 'ð', name: 'thumbs down' },
  { emoji: 'ð', name: 'clap' },
  { emoji: 'ð', name: 'raised hands' },
  { emoji: 'ð¤', name: 'handshake' },
  { emoji: 'âï¸', name: 'peace' },
  { emoji: 'ð¤', name: 'fingers crossed' },
  { emoji: 'ðª', name: 'muscle' },
  { emoji: 'ð«¶', name: 'heart hands' },
  { emoji: 'â¤ï¸', name: 'heart' },
  { emoji: 'ð¥', name: 'fire' },
  { emoji: 'ð', name: 'skull' },
  { emoji: 'ð©', name: 'poop' },
  { emoji: 'ð', name: 'eyes' },
  { emoji: 'ð', name: 'party' },
  { emoji: 'ð¿', name: 'popcorn' },
  { emoji: 'ð¬', name: 'clapper' },
  { emoji: 'ð½ï¸', name: 'film projector' },
  { emoji: 'ð', name: 'pizza' },
  { emoji: 'ðº', name: 'beer' },
  { emoji: 'â', name: 'coffee' },
  { emoji: 'ð¤¦', name: 'facepalm' },
  { emoji: 'ð¤·', name: 'shrug' },
  { emoji: 'ð¯', name: '100' },
  { emoji: 'â¨', name: 'sparkles' },
  { emoji: 'ð', name: 'laughing' },
  { emoji: 'ð', name: 'sweat smile' },
  { emoji: 'ð« ', name: 'melting' },
  { emoji: 'ð¥±', name: 'yawning' },
  { emoji: 'ð', name: 'neutral' },
];

// Ordered longest-first so e.g. :'( matches before :(
const EMOTICONS = [
  [":'(", 'ð¢'],
  [">:(", 'ð '],
  ['</3', 'ð'],
  [':D',  'ð'],
  [':P',  'ð'],
  [':p',  'ð'],
  [';)',  'ð'],
  [':)',  'ð'],
  [':]',  'ð'],
  [':(',  'ð'],
  [':[',  'ð'],
  [':|',  'ð'],
  [':/',  'ð'],
  [':o',  'ð®'],
  [':O',  'ð®'],
  [':*',  'ð'],
  ['B)',  'ð'],
  ['<3',  'â¤ï¸'],
  ['XD',  'ð'],
  ['xD',  'ð'],
  ['^_^', 'ð'],
  ['^^',  'ð'],
  ['-_-', 'ð'],
  ['o_o', 'ð³'],
  ['O_O', 'ð³'],
];

// Build a regex that matches any emoticon as a whole word/token
const EMOTICON_PATTERN = new RegExp(
  '(' + EMOTICONS.map(([e]) => e.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')).join('|') + ')',
  'g',
);

const EMOTICON_MAP = Object.fromEntries(EMOTICONS);

function replaceEmoticons(text) {
  return text.replace(EMOTICON_PATTERN, (match) => EMOTICON_MAP[match] || match);
}

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
      const text = replaceEmoticons(this.messageToBeSent).trim();
      if (text === '') return;
      this.SEND_MESSAGE(text);
      this.messageToBeSent = '';
    },

    convertEmoticons() {
      const converted = replaceEmoticons(this.messageToBeSent);
      if (converted !== this.messageToBeSent) {
        const input = this.$refs.messageInput.$el.querySelector('input');
        const pos = input?.selectionStart ?? converted.length;
        this.messageToBeSent = converted;
        this.$nextTick(() => {
          input?.setSelectionRange(pos, pos);
        });
      }
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
