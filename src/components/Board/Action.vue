<template lang="pug">
.qkb-board-action(
  :class="actionClass"
)
  .qkb-board-action__wrapper
    slot(name="actions")
    button.qkb-action-item.qkb-action-item--audio#textToAudioId(@click="audioSwitch")
      slot(name="audioButton")
        IconAudio.qkb-action-icon.qkb-action-icon--audio
    .qkb-board-action__msg-box
      input.qkb-board-action__input#qkb-board-action__input(
        type="text",
        v-model="messageText",
        ref="qkbMessageInput",
        :disabled="inputDisable",
        :placeholder="inputPlaceholder",
        @keydown.enter="sendMessage",
      )
      .qkb-board-action__disable-text(
        v-if="inputDisablePlaceholder && inputDisable"
      )
        span {{ inputDisablePlaceholder }}
    .qkb-board-action__extra
      slot(name="actions")
      button.qkb-action-item.qkb-action-item--microphone(@click="audioToText")
        slot(name="microphoneButton")
          IconMicrophone.qkb-action-icon.qkb-action-icon--microphone
      button.qkb-action-item.qkb-action-item--send(@click="sendMessage")
        slot(name="sendButton")
          IconSend.qkb-action-icon.qkb-action-icon--send
</template>
<script>
import IconSend from '../../assets/icons/send.svg'
import IconMicrophone from '../../assets/icons/microphone.svg'
import IconAudio from '../../assets/icons/audio.svg'
import Vue from 'vue'

export default {

  components: {
    IconMicrophone,
    IconSend,
    IconAudio
  },

  props: {
    inputPlaceholder: {
      type: String
    },

    inputDisablePlaceholder: {
      type: String
    },

    inputDisable: {
      type: Boolean,
      default: false
    }
  },

  data () {
    return {
      messageText: null,
      runtimeTranscription_: '',
      transcription_: [],
      lang_: 'hi-IN',
      pitch: 1,
      rate: 1,
      synth: window.speechSynthesis,
      validation: false,
      audioout: false
    }
  },

  computed: {
    actionClass () {
      const actionClasses = []

      if (this.inputDisable) {
        actionClasses.push('qkb-board-action--disabled')
      }

      if (this.messageText) {
        actionClasses.push('qkb-board-aciton--typing')
      }

      return actionClasses
    }
  },

  mounted () {
    this.$refs.qkbMessageInput.focus()
  },

  created () {
    this.$root.$refs.Action = this
  },

  methods: {
    sendMessage () {
      if (this.messageText) {
        this.$emit('msg-send', { text: this.messageText })
        this.messageText = null
      }
    },
    audioToText () {
      window.SpeechRecognition = window.webkitSpeechRecognition
      const recognition = new window.SpeechRecognition()
      recognition.lang = this.lang_
      recognition.interimResults = true

      // event current voice record word
      recognition.addEventListener('result', event => {
        var text = Array.from(event.results)
          .map(result => result[0])
          .map(result => result.transcript)
          .join('')
        this.messageText = text
      })
      // end of transcription
      recognition.addEventListener('end', () => {
        this.transcription_.push(this.runtimeTranscription_)
        recognition.stop()
        setTimeout(() => {
          if (this.messageText) {
            this.$emit('msg-send', { text: this.messageText })
            this.messageText = null
          }
        }, 2000)
      })
      recognition.start()
    },
    audioSwitch () {
      if (Vue.prototype.$audioOut) {
        Vue.prototype.$audioOut = false
      } else {
        Vue.prototype.$audioOut = true
      }
    },
    textToAudio (botMessage) {
      if (botMessage) {
        this.validation = false
        let sInstance = new SpeechSynthesisUtterance(botMessage)
        sInstance.lang = this.lang_

        sInstance.onend = function (event) {
        }
        sInstance.onerror = function (event) {
        }
        sInstance.pitch = this.pitch
        sInstance.rate = this.rate
        this.synth.speak(sInstance)
      } else {
        this.validation = true
      }
    }
  }
}

</script>
