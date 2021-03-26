<template>
<div>
  <div class="video-list" v-show="screenType==='live'">
    <div v-for="video in videoList" :key="video.id" :class="'video-item-'+video.role" v-show="video.openVideo">
      <video :id="video.id" controls autoplay playsinline ref="videos" :height="cameraHeight" :muted="video.muted"></video>
    </div>
  </div>
  <div class="design-w" id="designW" v-show="screenType==='design'">
    <div id="widget-container" style="position:absolute;bottom: 0;right: 0;left: 0;height: 100%;border: 1px solid black; border-top:0; border-bottom: 0;"></div>
  </div>
  <canvas id="temp-stream-canvas" style="display: none;"></canvas>
</div>
</template>

<script>
// eslint-disable-next-line
import * as io from 'socket.io-client';
import * as RTCMultiConnection from 'rtcmulticonnection';
import CanvasDesigner from '@/libs/canvas-designer-widget.js';
// eslint-disable-next-line
window.io = io;
export default {
  name: 'Rtc',
  data() {
    return {
      designer: null,
      rtcmConnection: null,
      videoList: [],
      canvas: null,
      temp: true
    };
  },
  props: {
    screenType: {
      type: String,
      default: ''
    },
    isTeacher: {
      type: Boolean,
      default() {
        return false;
      }
    },
    roomId: {
      type: String,
      default: 'public-room'
    },
    userInfo: {
      type: Object,
      default() {
        return {};
      }
    },
    socketURL: {
      type: String,
      default: 'https://192.168.50.31:9001/'
    },
    cameraHeight: {
      type: [Number, String],
      default: 160
    },
    autoplay: {
      type: Boolean,
      default: true
    },
    screenshotFormat: {
      type: String,
      default: 'image/jpeg'
    }
  },
  watch: {},
  mounted() {
    // fix消息发送的bug提示
    window.FileBufferReader = function() {};
    this.rtcmConnection = new RTCMultiConnection();
    this.rtcmConnection.socketURL = this.socketURL;
    // 通过extra附带一些基本信息
    this.rtcmConnection.extra.userFullName = this.userInfo.name;
    this.rtcmConnection.extra.name = this.userInfo.name;
    this.rtcmConnection.extra.id = this.userInfo.id;
    this.rtcmConnection.extra.role = this.userInfo.role;
    this.rtcmConnection.extra.openVideo = this.userInfo.openVideo;
    // 禁止自动创建元素
    this.rtcmConnection.autoCreateMediaElement = false;
    // this.rtcmConnection.publicRoomIdentifier = 'dashboard';
    // 两边通信需要一致
    this.rtcmConnection.socketMessageEvent = 'canvas-dashboard-demo';
    // 不需要打印log
    this.rtcmConnection.enableLogs = false;
    // 链接的内容，这里有视频、音频和message
    this.rtcmConnection.session = {
      audio: true,
      video: true,
      data: true
    };
    this.rtcmConnection.onstream = event => {
      console.log(event);
      console.log(this.videoList);
      if (!this.videoList.find(item => item.id === event.streamid) && this.videoList.length <= 8) {
        this.videoList.push({
          id: event.streamid,
          userId: event.extra.id,
          name: event.extra.name,
          role: event.extra.role,
          muted: true,
          openVideo: event.extra.openVideo
        });
        this.$emit('join', this.videoList);
        setTimeout(() => {
          this.$refs.videos.find(video => video.id === event.streamid).srcObject = event.stream;
        }, 100);
      }
    };
    this.rtcmConnection.onstreamended = event => {
      let index = this.videoList.findIndex(item => item.id === event.streamid);
      if (index !== -1) {
        this.videoList.splice(index, 1);
        this.$emit('join', this.videoList);
      }
    };
    this.rtcmConnection.onmessage = event => {
      console.log('event' + event.data);
      // 监听消息类型，不同消息做对应的处理
      if (event.data.connectingMike) {
        this.$emit('message', event.data.connectingMike, 'connectingMike');
      } else if (event.data.disConnectingMike) {
        this.$emit('message', event.data.disConnectingMike, 'disConnectingMike');
      } else if (event.data.type) {
        this.$emit('message', event.data.type, 'changeScreen');
      } else {
        this.$emit('message', event.data.message, 'message');
      }

      if (event.data === 'plz-sync-points') {
        this.designer.sync();
        return;
      }

      this.designer.syncData(event.data);
    };
    this.rtcmConnection.onopen = event => {
      // connection.onUserStatusChanged(event);

      if (this.designer.pointsLength <= 0) {
        // make sure that remote user gets all drawings synced.
        setTimeout(() => {
          this.rtcmConnection.send('plz-sync-points');
        }, 1000);
      }

      // document.getElementById('btn-chat-message').disabled = false;
      // document.getElementById('btn-attach-file').style.display = 'inline-block';
      // document.getElementById('btn-share-screen').style.display = 'inline-block';
    };
    this.createDesign();
  },
  methods: {
    join() {
      this.rtcmConnection.openOrJoin(this.roomId, (isRoomJoined, roomid, error) => {
        console.log(isRoomJoined, roomid, error);
      });
    },
    sendMessage(message) {
      this.rtcmConnection.send({
        message
      });
    },
    fullScreen(screen) {
      console.log('fullScreen');
      if (screen === '全屏') {
        document.getElementById('designW').style.width = document.documentElement.clientWidth + 'px';
        document.getElementById('designW').style.height = document.documentElement.clientHeight + 'px';
      } else {
        document.getElementById('designW').style.width = '800px';
        document.getElementById('designW').style.height = '340px';
      }
    },
    disConnectingMike(disConnectingMike, isFirst) {
      console.log(disConnectingMike);
      this.videoList.map(item => {
        if (disConnectingMike.id === item.userId) {
          item.openVideo = false;
        }
      });
      if (isFirst) {
        this.rtcmConnection.send({
          disConnectingMike
        });
      }
      console.log(this.videoList);
    },
    connectingMike(connectingMike, isFirst) {
      console.log(connectingMike);
      // 学生id为连麦id时显示学生video
      this.videoList.map(item => {
        if (connectingMike.id === item.userId) {
          item.openVideo = true;
        }
      });
      // 如果未连麦，发送连麦消息
      if (isFirst) {
        this.rtcmConnection.send({
          connectingMike
        });
      }
      console.log(this.videoList);
    },
    changeScreen(type, isFirst) {
      console.log(type);
      if (isFirst) {
        this.rtcmConnection.send({
          type
        });
      }
      document.getElementById('designW').style.width = '800px';
      document.getElementById('designW').style.height = '340px';
    },
    leave() {
      this.rtcmConnection.attachStreams.forEach(localStream => {
        localStream.stop();
      });
      this.videoList = [];
    },
    addStreamStopListener(stream, callback) {
      let streamEndedEvent = 'ended';
      if ('oninactive' in stream) {
        streamEndedEvent = 'inactive';
      }
      stream.addEventListener(
        streamEndedEvent,
        function() {
          callback();
          callback = function() {};
        },
        false
      );
    },
    onGettingSteam(stream) {
      this.rtcmConnection.addStream(stream);
      this.$emit('share-started', stream.streamid);
      let that = this;
      this.addStreamStopListener(stream, function() {
        console.log('share-stopped');
        that.rtcmConnection.removeStream(stream.streamid);
        that.$emit('share-stopped', stream.streamid);
      });
    },
    getDisplayMediaError(error) {
      console.log('Media error: ' + JSON.stringify(error));
    },
    shareScreen() {
      let that = this;
      if (navigator.getDisplayMedia || navigator.mediaDevices.getDisplayMedia) {
        if (navigator.mediaDevices.getDisplayMedia) {
          navigator.mediaDevices
            .getDisplayMedia({ video: true, audio: false })
            .then(stream => {
              that.onGettingSteam(stream);
            }, that.getDisplayMediaError)
            .catch(that.getDisplayMediaError);
        } else if (navigator.getDisplayMedia) {
          navigator
            .getDisplayMedia({ video: true })
            .then(stream => {
              that.onGettingSteam(stream);
            }, that.getDisplayMediaError)
            .catch(that.getDisplayMediaError);
        }
      }
    },
    createDesign() {
      console.log(123);
      // eslint-disable-next-line
      let designer = new CanvasDesigner();
      designer.widgetHtmlURL = './widget.html';
      designer.widgetJsURL = './widget.js';
      this.designer = designer;
      designer.addSyncListener(data => {
        this.rtcmConnection.send(data);
      });
      if (this.isTeacher) {
        designer.setSelected('pencil');
        designer.setTools({
          pencil: true,
          text: true,
          image: true,
          pdf: true,
          eraser: true,
          line: true,
          arrow: true,
          dragSingle: true,
          dragMultiple: true,
          arc: true,
          rectangle: true,
          quadratic: false,
          bezier: true,
          marker: true,
          zoom: false,
          lineWidth: false,
          colorsPicker: false,
          extraOptions: false,
          code: false,
          undo: true
        });
      } else {
        designer.setSelected('pdf');
        designer.setTools({
          pencil: false,
          text: false,
          image: false,
          pdf: false,
          eraser: false,
          line: false,
          arrow: false,
          dragSingle: false,
          dragMultiple: false,
          arc: false,
          rectangle: false,
          quadratic: false,
          bezier: false,
          marker: false,
          zoom: false,
          lineWidth: false,
          colorsPicker: false,
          extraOptions: false,
          code: false,
          undo: false
        });
      }
      designer.appendTo(document.getElementById('widget-container'), () => {
        if (this.isTeacher) {
          // let tempStreamCanvas = document.getElementById('temp-stream-canvas');
          // let tempStream = tempStreamCanvas.captureStream();
          // tempStream.isScreen = true;
          // tempStream.streamid = tempStream.id;
          // tempStream.type = 'local';
          // this.rtcmConnection.attachStreams.push(tempStream);
          // window.tempStream = tempStream;
          // this.rtcmConnection.extra.roomOwner = true;
          // this.rtcmConnection.open(this.roomId, (isRoomOpened, roomid, error) => {
          //   if (error) {
          //     if (error === this.rtcmConnection.errors.ROOM_NOT_AVAILABLE) {
          //       alert('Someone already created this room. Please either join or create a separate room.');
          //       return;
          //     }
          //     alert(error);
          //   }
          //   this.rtcmConnection.socket.on('disconnect', function() {
          //     location.reload();
          //   });
          // });
        } else {
          // this.rtcmConnection.join(this.roomId, (isRoomJoined, roomid, error) => {
          //   if (error) {
          //     if (error === this.rtcmConnection.errors.ROOM_NOT_AVAILABLE) {
          //       alert('This room does not exist. Please either create it or wait for moderator to enter in the room.');
          //       return;
          //     }
          //     if (error === this.rtcmConnection.errors.ROOM_FULL) {
          //       alert('Room is full.');
          //       return;
          //     }
          //     if (error === this.rtcmConnection.errors.INVALID_PASSWORD) {
          //       this.rtcmConnection.password = prompt('Please enter room password.') || '';
          //       if (!this.rtcmConnection.password.length) {
          //         alert('Invalid password.');
          //         return;
          //       }
          //       this.rtcmConnection.join(this.roomId, function(isRoomJoined, roomid, error) {
          //         if (error) {
          //           alert(error);
          //         }
          //       });
          //       return;
          //     }
          //     alert(error);
          //   }
          //   this.rtcmConnection.socket.on('disconnect', function() {
          //     location.reload();
          //   });
          // });
        }
      });
    }
  }
};
</script>
<style lang="scss">
.video-list {
  width: 800px;
  height: 340px;
  background-color: #000;
  border-radius: 8px;
  flex-wrap: wrap;
  .video-item-student {
    width: 200px;
    height: 100px;
    float: right video {
      width: 100%;
      height: 100%;
    }
  }
  .video-item-teacher {
    width: 400px;
    height: 170px;
    margin: 0 auto;
    video {
      width: 100%;
      height: 100%;
    }
  }
}
.design-w {
    width: 100vw;
    height: 100vh;
    position: relative;
    left: 0px;
    z-index: 10;
  }
</style>
