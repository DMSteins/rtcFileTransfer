<template>
  <button @click="openChannel">创建offer</button>
  <div>
    <div>
      <div>
        <textarea type="text" name="" v-model="text" />
        <button @click="openAnswer">创建answer</button>
      </div>
      <div>
        <input type="text" name="" v-model="msg" />
        <button @click="send">发送消息</button>
      </div>
    </div>
    <div>

    </div>
  </div>


</template>

<script setup lang="ts">
import { onMounted, reactive, ref } from 'vue'
const config = {
  'sdpSemantics': 'unified-plan',
  'iceServers': [{
    urls: 'stun:stun.l.google.com:19302'
  }]
}
const text = ref("")
const msg = ref("")
let conn: RTCPeerConnection | null = null
const openConnect = () => {
  conn = new RTCPeerConnection(config)
  conn.onconnectionstatechange = (e) => {
    console.log("onconnectionstatechange: ", e)
  }
  conn.onicecandidate = (e) => {
    console.log("onicecandidate: ", e)
  }
  conn.oniceconnectionstatechange = (e) => {
    console.log("oniceconnectionstatechange: ", e)
  }
  conn.ondatachannel = onChannelOpened
}
let _channel: RTCDataChannel | null = null
const openChannel = () => {
  if (!conn) return
  const channel = conn.createDataChannel('data-channel');
  channel.onopen = onChannelOpened
  conn.createOffer().then((d) => {
    if (!conn) return
    console.log('onDescription: ', d)
    conn.setLocalDescription(d)
  }).catch(e => {
    console.log('createOffer error:', e)
  });
}
const openAnswer = () => {
  if (!conn) return
  conn.setRemoteDescription(new RTCSessionDescription({ sdp: text.value, type: 'offer' }))
    .then(_ => {
      if (!conn) return
      return conn.createAnswer()
        .then(d => {
          if (!conn) return
          conn.setLocalDescription(d)
          console.log('answer onDescription: ', d)
        });

    })
    .catch(e => {
      console.log('answer error:', e)
    });
}

const onChannelOpened = (event) => {
  console.log('**********RTC: channel opened with', event);
  const channel = event.channel || event.target;
  channel.binaryType = 'arraybuffer';
  channel.onmessage = (e) => onMsg(e.data);
  channel.onclose = (e) => onClose();
  _channel = channel
}
const messages = reactive<string[]>([])
const onMsg = (msgdata: string) => {
  console.log(msgdata)
  // messages.push(msgdata)
}
const onClose = () => {
  console.log("channel close, should reopen")
}
const send = () => {
  console.log(_channel)
  if (!_channel) return;
  _channel.send(msg.value)
}
onMounted(() => {
  if (!conn) {
    openConnect()
  }
})
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
