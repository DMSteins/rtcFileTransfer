<template>
  <div class="box">
    <div class="peer-item">
      <div>发起者</div>
      <button @click="createOfferPeer">创建offer peer</button>
      <div>
        <!-- <div>
          <textarea type="text" name="" v-model="text" />
          <button >设置远程answer desc</button>
        </div> -->
        <div>
          <input type="text" name="" v-model="offerInput" />
          <button @click="answerSend">发送消息</button>
        </div>
      </div>
    </div>
    <div class="peer-item">
      <div>响应者</div>
      <button @click="createAnswerPeer">创建answer peer</button>
      <div>
        <div>
          <textarea type="text" name="" v-model="connID" />
          <button @click="answerConnect">连接offer</button>
        </div>
        <div>
          <input type="text" name="" v-model="answerInput" />
          <button @click="answerSend">发送消息</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, reactive, ref } from 'vue'
import { Peer } from "peerjs";
const offerInput = ref('')
const answerInput = ref('')
const connID = ref('')
let isCaller = false
let peer: Peer
let conn
const createOfferPeer = () => {
  isCaller = true
  initialize()
}
const createAnswerPeer = () => {
  initialize()
}
const answerConnect = () => {
  if (conn) {
    conn.close()
  }
  conn = peer.connect(connID.value, { reliable: true })
  conn.on('open', () => {
    console.log('open:', conn.peer)
  })
  conn.on('data', (data) => {
    console.log('answer get msg:', data)
  })
  conn.on('close', () => {

  })
}
const answerSend = () => {
  if (conn && conn.open) {
    conn.send(answerInput.value || offerInput.value)
  }
}
const initialize = () => {
  // Create own peer object with connection to shared PeerJS server
  peer = new Peer();

  peer.on('open', function (id) {
    // Workaround for peer.reconnect deleting previous id
    // if (peer.id === null) {
    //   console.log('Received null id from peer open');
    //   peer.id = lastPeerId;
    // } else {
    //   lastPeerId = peer.id;
    // }

    console.log('ID: ', peer.id);
    // recvId.innerHTML = "ID: " + peer.id;
    // status.innerHTML = "Awaiting connection...";
  });
  peer.on('connection', function (c) {
    // Allow only a single connection
    // if (conn && conn.open) {
    //   c.on('open', function () {
    //     c.send("Already connected to another client");
    //     setTimeout(function () { c.close(); }, 500);
    //   });
    //   return;
    // }
    if (isCaller) {
      conn = c;
      conn.on('data', (data) => {
        console.log('caller get msg:', data)
      })
    }

    console.log("Connected to: " + conn.peer);

  });
  peer.on('disconnected', function () {

    console.log('Connection lost. Please reconnect');

    // // Workaround for peer.reconnect deleting previous id
    // peer.id = lastPeerId;
    // peer._lastServerId = lastPeerId;
    // peer.reconnect();
  });
  peer.on('close', function () {
    conn = null;
    console.log('Connection destroyed');
  });
  peer.on('error', function (err) {
    console.log(err);
    alert('' + err);
  });
};
onMounted(() => {

})
</script>

<style>
.box {
  display: flex;
}

.peer-item {
  width: 50%;
}
</style>
