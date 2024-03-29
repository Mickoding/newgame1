<template>
    <v-card class="emoji-wheel">
      <v-row justify="center">
        <v-col v-for="(emoji, index) in emojis" :key="index" cols="auto">
          <v-btn @click="selectEmoji(emoji)">{{ emoji }}</v-btn>
        </v-col>
      </v-row>
    </v-card>
  </template>
  
  <script>
  export default {
    data() {
      return {
        emojis: ['😀', '😁', '😂', '🤣', '😃', '😄', '😅', '😆', '😉', '😊', '😋', '😎', '😍', '😘', '🥰', '😗', '😙', '😚', '🙂', '🤗']
      };
    },
    methods: {
      selectEmoji(emoji) {
        this.$emit('emoji-selected', emoji);
      }
    }
  };
  </script>
  
  <style scoped>
  .emoji-wheel {
    position: absolute;
    bottom: 10px;
    left: 50%;
    transform: translateX(-50%);
  }
  </style>