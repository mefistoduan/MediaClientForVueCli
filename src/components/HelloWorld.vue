<template>
  <div class="hello">
    <h5>mediasoup</h5>
    <div id="login" v-if="show == 1">
      Room: <input id="roomidInput" v-model="roomidInput" type="text"/>
      <br/>
      <br/>
      User: <input id='nameInput' v-model="nameInput" type="text"/>
      <br>
      <br>
      <button id='joinButton' @click="joinRoom()">Join</button>
    </div>
    <div id="control" class="hidden" v-if="show == 2">
      <button id='exitButton' @click="exit()">Exit</button>
      <br/>
      <br/>
      audio: <select id="audioSelect" v-model="audioSelect">
      <option v-for="audio in audioOptions" :value="audio.deviceId" :label="audio.label"></option>
    </select>
      <br/>
      <br/>
      video: <select id="videoSelect" v-model="videoSelect">
      <option v-for="video in videoOptions" :value="video.deviceId" :label="video.label"></option>
    </select>
      <br/>
      <br/>
      <button id='startAudioButton' @click="produce('audio')" v-if="btnAudioShow">
        audio
      </button>
      <button id='stopAudioButton' @click="closeProducer('audio')" v-if="!btnAudioShow">close
        audio
      </button>
      &nbsp;
      <button id='startVideoButton' @click="produce('video')" v-if="btnVideoButton">
        video
      </button>
      <button id='stopVideoButton' @click="closeProducer('video')" v-if="!btnVideoButton">close
        video
      </button>
      &nbsp;
      <button id='startScreenButton' @click="produce('screen')" v-if="btnScreenButton">screen</button>
      <button id='stopScreenButton' @click="closeProducer('screen')" v-if="!btnScreenButton">close
        screen
      </button>
      <br/>
    </div>
    <div id='videoMedia' v-show="show == 2">
      <h2>------local------</h2>
      <div id="localMedia"></div>
      <!--      <video id="localVideo" autoplay inline class="vid" :srcObject="localVideo.srcObject" :playsinline="false"></video>-->
      <!--      <video id="localScreen" autoplay inline class="vid"></video>-->
      <h2>-----remote-----</h2>
      <div id="remoteVideos" class="container">
      </div>

      <div id="remoteAudios"></div>
    </div>
  </div>
</template>

<script>
import socket from '../static/socket.io'
import RoomClient from '../static/RoomClient'
import mediasoupclient from '../static/mediasoupclient.min'

const socketIo = socket('https://192.168.0.91:3016')
socketIo.request = function request(type, data = {}) {
  return new Promise((resolve, reject) => {
    socketIo.emit(type, data, (data) => {
      if (data.error) {
        reject(data.error)
      } else {
        resolve(data)
      }
    })
  })
}

export default {
  data() {
    return {
      roomidInput: '123',
      nameInput: 'bob' + parseInt(Math.random() * 100),
      show: 1,
      RoomClient: null,
      btnAudioShow: true,
      btnVideoButton: true,
      btnScreenButton: true,
      producer: null,
      audioOptions: [],
      videoOptions: [],
      audioSelect: '',
      videoSelect: '',
      localVideo: {
        srcObject: null
      },
    }
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      let that = this;
      //   获取摄像头
      navigator.mediaDevices.enumerateDevices().then(devices =>
          devices.forEach(device => {
            if ('audioinput' === device.kind) {
              that.audioOptions.push(device);
              that.audioSelect = that.audioOptions[0].deviceId;//默认第一个
            } else if ('videoinput' === device.kind) {
              that.videoOptions.push(device);
              that.videoSelect = that.videoOptions[0].deviceId;//默认第一个
            }
          })
      )
    },
    joinRoom() {
      let that = this;
      that.show = 2;
      if (that.rc && that.rc.isOpen()) {
        console.log('already connected to a room')
      } else {
        let localMedia = document.getElementById('localMedia');
        let remoteVideos = document.getElementById('remoteVideos');
        let remoteAudios = document.getElementById('remoteAudios');
        let socket = socketIo;
        console.log(socket);
        let room_id = this.roomidInput;
        let name = this.nameInput;
        let roomOpen = function () {
          console.log('链接已建立');
        }
        // 创建RoomClient链接
        that.rc = new RoomClient(localMedia, remoteVideos, remoteAudios, window.mediasoupClient, socket, room_id, name, roomOpen, that)
      }
    },
    addListeners() {
      // 转由vue实现dom元素的切换
      let that = this;
      rc.on(RoomClient.EVENTS.startScreen, () => {
        that.btnAudioShow = false;
      })
    },
    exit() {
      this.show = 1;
      // todo
      this.rc.exit();
      console.log('disconnect：', '已经断开 socket 链接');
    },
    produce(type) {
      let that = this;
      console.log(this.rc);
      if (type == 'audio') {
        this.rc.produce(RoomClient.mediaType.audio, audioSelect.value)
      }
      if (type == 'video') {
        console.log(123);
        this.rc.produce(RoomClient.mediaType.video, videoSelect.value)
      }
      if (type == 'screen') {
        this.rc.produce(RoomClient.mediaType.screen)
      }
    },
    closeProducer() {
      // todo
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

#localMedia {

  display: block;
  overflow: hidden;
}

/deep/ .vid {
  width: 640px;
  height: 320px;
  overflow: hidden;
  float: left;
}
</style>
