<template lang="pug">
.qkb-board-content(ref="boardContent")
  .qkb-board-content__bubbles(
    ref="boardBubbles"
  )
    message-bubble(
      v-for="(item, index) in mainData",
      :key="index",
      :message="item",
      :class="audiocaller"
    )
    .qkb-board-content__bot-typing(v-if="botTyping")
      slot(name="botTyping")
        message-typing
</template>

<script>
import MessageBubble from '../MessageBubble/Main'
import MessageTyping from '../MessageBubble/Typing'
import Vue from 'vue'

var counter = new Vue({
  data: {
    messageCounter: 0,
    audioout: false
  }
})

export default {
  components: {
    MessageBubble,
    MessageTyping,
    counter
  },

  props: {
    mainData: {
      type: Array,
      required: true
    },

    botTyping: {
      type: Boolean,
      default: false
    }
  },

  watch: {
    mainData: function (newVal) {
      this.$nextTick(() => {
        this.updateScroll()
      })
    }
  },

  computed: {
    audiocaller () {
      if (this.mainData.length > counter.messageCounter) {
        if (Vue.prototype.$appName) {
          if (this.mainData[counter.messageCounter].agent === 'bot') {
            this.$root.$refs.Action.textToAudio(this.mainData[counter.messageCounter].text)
          }
        }
        counter.messageCounter++
      }
      return ''
    }
  },

  methods: {
    updateScroll () {
      const contentElm = this.$refs.boardContent
      const offsetHeight = this.$refs.boardBubbles.offsetHeight
      contentElm.scrollTop = offsetHeight
    }
  }
}
</script>
