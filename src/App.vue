<template>
  <div id="app" :class="{ 'has-background-dark': darkMode }">
    <section class="section">
      <div class="container">
        <div class="header">
          <h1 class="title" :class="{ 'has-text-white': darkMode }">Textractor WebSocket Client</h1>
          <dark-mode-toggle @dark-mode-toggle="handleDarkModeToggle"></dark-mode-toggle>
        </div>
        <button class="button is-primary" v-if="buttonConnectVisibility" :disabled="buttonDisabled"
          @click="prompt">Connect</button>
        <button class="button is-danger" v-if="!buttonConnectVisibility" @click="disconnect()">Disconnect</button>
        <button class="button is-warning" @click="clear()">Clear</button>
      </div>
    </section>
    <section class="messages">
      <div class="container">
        <div class="card fade-in-fwd" :class="{ 'has-background-grey-dark': darkMode }" v-for="(msg, index) in messages"
          :key='msg + "_" + index'>
          <div class="card-content" :class="{ 'has-text-white': darkMode }">
            {{ msg }}
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import DarkModeToggle from './components/DarkModeToggle.vue'

const CONNTECTION_STATUS = {
  DISCONNTECT: 0,
  CONNECTING: 1,
  CONNECTED: 2
};
const MAX_AMOUNT = 50;

export default {
  name: 'app',
  data: function () {
    return {
      darkMode: false,
      savedAddr: '',
      status: CONNTECTION_STATUS.DISCONNTECT,
      ws: null,
      messages: []
    }
  },
  components: {
    DarkModeToggle
  },
  mounted() {
    // Retrieve dark mode state from localStorage on component mount
    let savedAddr = localStorage.getItem('savedAddr')
    if (savedAddr != '' && savedAddr != null) {
      this.savedAddr = savedAddr
    } else {
      this.savedAddr = '127.0.0.1:1234'
    }
    this.darkMode = localStorage.getItem('darkMode') === 'true'
  },
  methods: {
    handleDarkModeToggle(darkMode) {
      this.darkMode = darkMode

      // Save dark mode state to localStorage
      localStorage.setItem('darkMode', darkMode.toString())
    },
    prompt() {
            this.$buefy.dialog.prompt({
                message: `Websocket Address?`,
                inputAttrs: {
                    placeholder: this.savedAddr,
                    value: this.savedAddr
                },
                trapFocus: true,
                onConfirm: (value) => this.connect(value)
            })
        },
    clear() {
      this.messages = []
      this.messages.length = 0;
    },
    connect(address) {
      localStorage.setItem('savedAddr', address)
      this.savedAddr = address
      this.status = CONNTECTION_STATUS.CONNECTING
      this.ws = new WebSocket('ws://' + address)
      this.ws.addEventListener('close', this.eventWebSocketClosed);
      this.ws.addEventListener('message', this.eventWebSocketMessage);
      this.ws.addEventListener('open', this.eventWebSocketOpen);
      this.ws.addEventListener('error', this.eventWebSocketError);
    },

    disconnect() {
      if (this.ws) {
        this.ws.close();
      }
    },

    eventWebSocketClosed() {
      this.status = CONNTECTION_STATUS.DISCONNTECT
      this.ws = null;
    },

    eventWebSocketMessage(event) {
      if(event.data === undefined) {return}
      this.messages.unshift(event.data);
      if (this.messages.length > MAX_AMOUNT)
        this.messages.pop();
    },

    eventWebSocketOpen() {
      this.status = CONNTECTION_STATUS.CONNECTED;
    },

    eventWebSocketError() {
      this.status = CONNTECTION_STATUS.DISCONNTECT;
    }
  },
  computed: {
    buttonConnectVisibility() {
      return this.status < CONNTECTION_STATUS.CONNECTED
    },

    buttonDisabled() {
      return this.status === CONNTECTION_STATUS.CONNECTING
    }
  }
}
</script>

<style>
#app,
#app .button {
  font-family: 'Kosugi Maru', sans-serif;
  min-height: 100%;
}

html,
body {
  height: 100%;
}

.header {
  display: flex;
  justify-content: space-between;
}

.card {
  font-size: 1.3em;
  margin-top: 1em;
}

.card:first-of-type {
  margin-top: 0em;
}

.button.is-warning {
  margin-left: 1em;
}

.fade-in-fwd {
  animation: fade-in-fwd 0.6s cubic-bezier(0.390, 0.575, 0.565, 1.000) both;
}

/* ----------------------------------------------
 * Generated by Animista on 2019-11-23 5:28:59
 * Licensed under FreeBSD License.
 * See http://animista.net/license for more info.
 * w: http://animista.net, t: @cssanimista
 * ---------------------------------------------- */

/**
 * ----------------------------------------
 * animation fade-in-fwd
 * ----------------------------------------
 */
@keyframes fade-in-fwd {
  0% {
    transform: translateZ(-80px);
    opacity: 0;
  }

  100% {
    transform: translateZ(0);
    opacity: 1;
  }
}</style>
