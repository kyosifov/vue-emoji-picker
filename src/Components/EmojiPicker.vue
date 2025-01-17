<template>
  <div>
    <slot
        name="emoji-invoker"
        :events="{ click: (e) => toggle(e) }"
    ></slot>
    <div
        v-if="display.visible"
    >
      <slot
          name="emoji-picker"
          :emojis="emojis"
          :insert="insert"
          :display="display"
          :hidePicker="hide"
      ></slot>
    </div>
  </div>
</template>

<script>
  import emojis from '../emojis';
  import { escapeRegex } from '../utils/helpers';

  export default {
    props: {
      search: {
        type: String,
        required: false,
        default: '',
      },
      emojiTable: {
        type: Object,
        required: false,
        default() {
          return emojis;
        },
      },
    },
    data() {
      return {
        display: {
          x: 0,
          y: 0,
          visible: false,
        },
      };
    },
    computed: {
      emojis() {
        if (this.search) {
          const obj = {};

          for (const category in this.emojiTable) {
            obj[category] = {};

            for (const emoji in this.emojiTable[category]) {
              if (new RegExp(`.*${escapeRegex(this.search)}.*`).test(emoji)) {
                obj[category][emoji] = this.emojiTable[category][emoji];
              }
            }

            if (Object.keys(obj[category]).length === 0) {
              delete obj[category];
            }
          }

          return obj;
        }

        return this.emojiTable;
      },
    },
    methods: {
      insert(emoji) {
        this.$emit('emoji', emoji);
      },
      toggle(e) {
        this.display.visible = !this.display.visible;
        this.display.x = e.clientX;
        this.display.y = e.clientY;
      },
      hide() {
        this.display.visible = false;
      },
      escape(e) {
        if (this.display.visible === true && e.keyCode === 27) {
          this.display.visible = false;
        }
      }
    },
    mounted() {
      document.addEventListener('keyup', this.escape);
    },
    destroyed() {
      document.removeEventListener('keyup', this.escape);
    },
  };
</script>
