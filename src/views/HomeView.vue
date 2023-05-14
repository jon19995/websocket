<template>
  <div class="container">
      <h1>Соединение:
        <span v-if="connection_ready" class="connection_ready">
          Установлено!
        </span>
      </h1>
      
      <div class="messages" id="messages">
        <div class="message-container">
          <h1 class="error" v-if="connection_error"> Ошибка! </h1>

          <div v-for="(m,idx) in messages" :key="'m-' + idx" style="clear:both">
            <div :class="{
                'msg-from-me': m.from === 'me',
                'msg-from-other': m.from === 'other'
              }"
            >
              {{m.message}}
            </div>
          </div> 
        </div>
      </div>

      <div class="send-zone">
        <input v-model="new_message" type="text" placeholder="Сообщение..." @keyup.enter="send_message"/>
        <button @click.prevent="send_message">Отправить</button>
      </div>
  </div>
</template>

<script>
export default {
  name: 'HomeView',
  data() {
    return {
      SOKET_URL: "javascript.info/article/websocket/demo/hello", 
      connection_ready: false,
      connection_error: false,
      nickname: "",
      websocket: null,
      new_message: "",
      messages: []
    }
  },
  mounted() {
    this.init_chat();
  },
  methods:{
    init_chat() {
      if (this.nickname === "") {
        this.nickname = prompt("Enter a nickname:");
      }

      const url = `wss://${this.SOKET_URL}`
      this.websocket = new WebSocket(url);

      this.websocket.onopen = this.onSocketOpen;
      this.websocket.onmessage = this.onSocketMessage;
      this.websocket.onerror = this.onSockerError;
    },

    onSocketOpen(evt){
      this.connection_ready = true;
    },

    onSocketMessage(evt){
      const received = JSON.parse(evt.data);
      this.messages.push( { from: "other", message: received.message } );
      const messages_div = document.getElementById('messages');
      messages_div.scrollTo({top: messages_div.scrollHeight, behavior: 'smooth'});
    },

    onSockerError(evt){
      this.connection_error = true;
    },

    send_message() {
      const to_send = { from: this.nickname, message: this.new_message };
      this.websocket.send( JSON.stringify(to_send) );
      this.messages.push( { from: "me", message: this.new_message } );
      this.new_message = "";
    }
  },
}
</script>

<style lang="less">
  body{
    background: #111b21;
  }
  .container{
    display: flex;
    flex-direction: column;
    margin: 0 auto;
    max-width: 768px;
    min-height: 98vh;
    position: relative;

    h1{
      padding: 0px;
      height: 20px;
      color: white;
      font-size: 20px;
      text-transform: uppercase;

      .connection_ready{
        color: greenyellow;
      }
    }

    .messages{
      height: 80vh;
      overflow-y: scroll;
      background: url(@/assets/background.jpg) no-repeat center;
      background-size: cover;
      
      

      .msg-from-me {
        border-radius: 7.5px;
        max-width: 65%;
        font-size: 16px;
        line-height: 19px;
        color: #e9edef;
        background: fade( #046a62, 90%);
        padding: 5px;
        margin: 20px 20px 5px 0px;
        
        float: right;
      }
      .msg-from-other {
        border-radius: 7.5px;
        max-width: 65%;
        font-size: 16px;
        line-height: 19px;
        color: #e9edef;
        background: fade( #202c33, 90%);
        padding: 5px;
        margin: 20px 0px 5px 20px;
        float: left;
      }
    }
    .send-zone{
      height: 62px;
      background: #202c33;
      display: flex;
      gap: 12px;

      input[type='text']{
        padding: 12px 20px;
        margin: 5px 0px;
        border: 1px solid #2a3942;
        background: #2a3942;
        border-radius: 8px;
        font-size: 15px;
        flex-grow: 1;
        color: white;
      }

      button {
        margin: 5px 0;
      }

    }
  }
  
</style>
