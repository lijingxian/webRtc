<template>
  <div class="live-detail-page">
    <div class="live-box">
      <div class="video-box">
        <Rtc
          ref="webrtc"
          width="100%"
          :screen-type="screenType"
          :isTeacher="true"
          :user-info="userInfo"
          :roomId="roomId"
          @join="onJoinUser"
          @message="onMessage"
        ></Rtc>
        <el-row>
          <el-button plain="plain" size="small" @click="changeScreen('live')"
            >直播屏</el-button
          >
          <el-button plain="plain" size="small" @click="changeScreen('design')"
            >课件屏</el-button
          >
          <el-button plain="plain" size="small" @click="onShareScreen"
            >共享屏幕</el-button
          >
          <el-button
            @click="startRecording"
            :disabled="videoStart"
            size="small"
            class="float-right"
            >开始录屏</el-button
          >
          <el-button
            @click="stopRecording"
            :disabled="!videoStart"
            size="small"
            class="float-right"
            >停止录屏</el-button
          >
          <el-button
            @click="fullScreen"
            v-show="screenType==='design'"
            size="small"
            class="float-right"
            >{{screen}}</el-button
          >
          <!-- <el-button class='float-right' plain='plain' size='small'>插入课件</el-button> -->
          <div class="clear"></div>
        </el-row>
      </div>
      <div class="chat-box">
        <div class="chat">
          <div class="chat-nav">
            <span
              class="nav"
              v-for="(item, index) in chatNav"
              :key="index"
              :class="{ active: navChoose == index }"
              @click="chooseNav(index)"
              >{{ item }}</span
            >
          </div>
          <div class="chose-nav" v-show="navChoose === 0">
            <div class="content-box">
              <div
                class="content"
                v-for="(item, index) in chatList"
                :key="index"
              >
                <span class="person-name">{{ item.name }}：</span>
                <span class="chat-content">{{ item.chat }}</span>
              </div>
            </div>
            <el-input
              class="input"
              v-model="chatInput"
              placeholder="请输入问题"
            ></el-input>
            <el-button type="warning" size="small" @click="sendMessage"
              >发送</el-button
            >
          </div>
          <div class="chose-nav" v-show="navChoose === 1">
            <p>{{ liveIntroduce }}</p>
          </div>
          <div class="chose-nav" v-show="navChoose === 2">
            <p>{{ teacherIntroduce }}</p>
          </div>
        </div>
        <el-row>
          <el-button plain="plain" size="small" @click="onJoin"
            >开始直播</el-button
          >
          <el-button
            class="float-right"
            plain="plain"
            size="small"
            @click="onLeave"
            >退出直播间</el-button
          >
          <div class="clear"></div>
        </el-row>
      </div>
      <video
        controls
        autoplay
        playsinline
        ref="video"
        style="display:none"
      ></video>
      <div class="student-box">
        <div class="title">学生列表</div>
        <div class="list-box">
          <el-popover
            v-for="item in studentList"
            placement="right"
            :key="item.id"
            width="86"
            v-model="item.visible"
          >
            <div>
              <p class="btn" @click="connectingMike(item)">连麦</p>
              <p class="btn" @click="disConnectingMike(item)">退出连麦</p>
              <p class="btn" @click="onDisable(item)">禁言</p>
              <p class="btn" @click="viewQue(item)">查看提问</p>
            </div>
            <div class="list" slot="reference">
              <img class="student-img" :src="item.imgUrl" />
              <p class="student-name" :class="{ 'is-link': item.isLink }">
                {{ item.name }}
              </p>
            </div>
          </el-popover>
        </div>
      </div>
      <div class="works">
        <div
          class="work"
          v-for="item in works"
          @click="toWork(item)"
          :key="item.id"
        >
          <img :src="item.pic" />
          <p>{{ item.name }}</p>
        </div>
      </div>
      <div class="evaluate">
        <!-- <Collapse title='课程评价'>
          <div slot='content'>
            <div class='collapse-row' v-for='item in evaluateList' :key='item.id'>
              <div class='name'>{{item.name}}:</div>
              <div>{{item.text}}</div>
            </div>
            <el-pagination
              class='pagination'
              layout='prev,pager,next,total,jumper'
              :hide-on-single-page='true'
              :total='total'
              :current-page='currentPage'
              :page-size='3'
              @current-change='handleCurrentChange'
              @prev-click='handlePrevClick'
              @next-click='handleNextClick'
            ></el-pagination>
          </div>
        </Collapse>-->
      </div>
    </div>
    <el-dialog :visible.sync="dialogVisible" title="查看提问">
      <div class="chose-nav" v-show="navChoose === 0">
        <div class="content-box">
          <div class="content" v-for="item in question" :key="item.id">
            <span class="person-name">{{ item.name }}：</span>
            <span class="chat-content">{{ item.content }}</span>
          </div>
        </div>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import pic from '@/assets/img/live/pic.png';
import Rtc from '@/components/Rtc';
import RecordRTC from 'recordrtc';
export default {
  name: 'Teacher',
  components: { Rtc },
  data() {
    return {
      screenType: 'design',
      video: null,
      videoStart: false,
      recorder: null,
      screen: '全屏',
      userInfo: {
        id: '123124314',
        name: '何老师',
        role: 'teacher',
        openVideo: true
      },
      img: null,
      dialogVisible: false,
      roomId: 'public-room',
      liveSrc: '/static/movie/test.mp4',
      chatNav: ['在线答疑', '直播简介', '教师简介'],
      navChoose: 0,
      question: [],
      chatList: [
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' },
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' },
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' },
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' },
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' },
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' }
      ],
      liveIntroduce:
        '直播简介直播简介直播简介直播简介直播简介直播简介直播简介直播简介直播简介直播简介',
      teacherIntroduce:
        '老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介',
      chatInput: '',
      studentList: [],
      works: [
        { pic: pic, name: '优秀作品', url: '' },
        { pic: pic, name: '优秀作品', url: '' },
        { pic: pic, name: '优秀作品', url: '' },
        { pic: pic, name: '优秀作品', url: '' },
        { pic: pic, name: '优秀作品', url: '' },
        { pic: pic, name: '优秀作品', url: '' }
      ],
      evaluateList: [
        {
          name: '王小明',
          text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！'
        },
        {
          name: '王小明',
          text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！'
        },
        {
          name: '王小明',
          text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！'
        },
        {
          name: '王小明',
          text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！'
        },
        {
          name: '王小明',
          text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！'
        },
        {
          name: '王小明',
          text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！'
        },
        {
          name: '王小明',
          text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！'
        },
        {
          name: '王小明',
          text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！'
        },
        {
          name: '王小明',
          text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！'
        },
        {
          name: '王小明',
          text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！'
        }
      ],
      total: 100,
      currentPage: 1
    };
  },
  computed: {},
  watch: {},
  created() {
    // this.connectRoom();
  },
  mounted() {
    this.video = document.querySelector('video');
    setTimeout(() => {
      this.screenType = 'live';
    }, 50);
  },
  methods: {
    changeScreen(type) {
      console.log(type);
      this.screenType = type;
      this.$refs.webrtc.changeScreen(type, true);
    },
    fullScreen() {
      this.$refs.webrtc.fullScreen(this.screen);
      if (this.screen === '全屏') {
        this.screen = '退出全屏';
      } else {
        this.screen = '全屏';
      }
    },
    onJoinUser(userList) {
      console.log(userList);
      this.studentList = [];
      for (const user of userList) {
        this.studentList.push({
          id: user.userId,
          name: user.name,
          visible: true
        });
      }
    },
    onCapture() {
      this.img = this.$refs.webrtc.capture();
    },
    onJoin() {
      this.$refs.webrtc.join();
    },
    onLeave() {
      this.$refs.webrtc.leave();
      this.studentList.map(item => {
        if (item.id === this.userInfo.id) {
          this.studentList.splice(item, 1);
        }
      });
    },
    onShareScreen() {
      this.img = this.$refs.webrtc.shareScreen();
    },
    onError(error, stream) {
      console.log('On Error Event', error, stream);
    },
    logEvent(event) {
      console.log('Event : ', event);
    },
    chooseNav(index) {
      this.navChoose = index;
    },
    toWork(item) {
      this.$router.push({ path: item.url });
    },
    handleCurrentChange(val) {
      console.log('当前页', val);
    },
    handlePrevClick() {
      this.currentPage--;
    },
    handleNextClick() {
      this.currentPage++;
    },
    viewQue(student) {
      this.question = [];
      console.log('提问' + student);
      console.log('student' + student.id);
      this.chatList.map(item => {
        console.log('item' + item.id);
        if (item.name === student.name) {
          this.question.push({ name: item.name, content: item.chat });
        }
      });
      this.dialogVisible = true;
    },
    sendMessage() {
      let message = {
        from: {
          role: 'teacher',
          id: '12345',
          name: '何老师'
        },
        chat_type: 'group',
        payload: {
          body: {
            msg: this.chatInput, // 消息内容
            type: 'txt' // 文本消息类型
          }
        }
      };
      this.$refs.webrtc.sendMessage(message);
      this.chatList.push({
        name: '何老师',
        isTeacher: true,
        chat: this.chatInput
      });
      this.chatInput = '';
    },
    onMessage(message) {
      console.log(message);
      if (message) {
        this.chatList.push({
          name: message.from.name,
          isTeacher: message.from.role === 'teacher',
          chat: message.payload.body.msg
        });
      }
    },
    connectingMike(item) {
      this.$refs.webrtc.connectingMike(item, true);
    },
    disConnectingMike(item) {
      this.$refs.webrtc.disConnectingMike(item, true);
    },
    onDisable(item) {
      console.log(item);
      // // 禁言
      let message = {
        from: {
          role: 'teacher',
          id: '12345',
          name: '何老师'
        },
        chat_type: 'disabled',
        payload: {
          body: {
            id: item.id,
            name: item.name,
            disabled: true
          }
        }
      };
      this.$refs.webrtc.sendMessage(message);
    },
    captureCamera(callback) {
      navigator.getDisplayMedia({ video: true })
        .then(function(camera) {
          callback(camera);
        })
        .catch();
    },
    getFileName(fileExtension) {
      let d = new Date();
      let year = d.getUTCFullYear();
      let month = d.getUTCMonth();
      let date = d.getUTCDate();
      return (
        'RecordRTC-' +
        year +
        month +
        date +
        '-' +
        this.getRandomString() +
        '.' +
        fileExtension
      );
    },
    getRandomString() {
      if (
        window.crypto &&
        window.crypto.getRandomValues &&
        navigator.userAgent.indexOf('Safari') === -1
      ) {
        let a = window.crypto.getRandomValues(new Uint32Array(3));
        let token = '';
        for (let i = 0, l = a.length; i < l; i++) {
          token += a[i].toString(36);
        }
        return token;
      } else {
        return (Math.random() * new Date().getTime())
          .toString(36)
          .replace(/\./g, '');
      }
    },
    stopRecordingCallback() {
      this.video.src = this.video.srcObject = null;
      this.video.muted = false;
      this.video.volume = 1;
      this.video.src = URL.createObjectURL(this.recorder.getBlob());
      window.open(URL.createObjectURL(this.recorder.getBlob()));
      // const a = document.createElement('a');
      // a.style.display = 'none';
      // a.download = '<文件名>';
      // a.href = this.video.src;
      // a.click();
      // let fileName = this.getFileName('webm');
      // // we need to upload 'File' --- not 'Blob'
      // let fileObject = new File([this.recorder.getBlob()], fileName, {
      //   type: 'video/webm'
      // });
      // let formData = new FormData();
      // // recorded data
      // formData.append('video-blob', fileObject);
      // // file name
      // formData.append('video-filename', fileObject.name);
      // let request = new XMLHttpRequest();
      // request.open('post', 'http://192.168.1.5:3200/upload');
      // request.send(formData);
      // request.onreadystatechange(() => {
      //   if (request.readyState === 4 && request.status === 200) {
      //     alert(request.responseText);
      //   } else {
      //     alert(request.statusText);
      //   }
      // });
      // this.recorder.camera.stop();
      // this.recorder.destroy();
      // this.recorder = null;
    },
    startRecording() {
      // console.log(this.video);
      this.videoStart = true;
      this.captureCamera(camera => {
        this.video.muted = true;
        this.video.volume = 0;
        this.video.srcObject = camera;
        this.recorder = RecordRTC(camera, {
          type: 'video'
        });
        this.recorder.startRecording();
        // release camera on stopRecording
        this.recorder.camera = camera;
      });
      // if (!navigator.mediaDevices || !navigator.getDisplayMedia) {
      //   console.log('不支采集音视频数据！');
      // } else {
      //   // 采集音频数据
      //   let constrants = {
      //     video: true
      //   };
      //   navigator.getDisplayMedia(constrants).then(this.gotMediaStream).catch(this.handleError);
      // }
    },
    gotMediaStream(stream) {
      console.log('数据流' + stream);
      this.video.srcObject = stream;
    },
    handleError(err) {
      console.log(err.name + ':' + err.message);
    },
    stopRecording() {
      this.videoStart = false;
      // window.open(URL.createObjectURL(this.video.srcObject.getBlob()));
      this.recorder.stopRecording(this.stopRecordingCallback);
    }
  }
};
</script>
<style lang='scss'>
.live-box {
  width: 1100px;
  margin: 0 auto;
  margin-top: 30px;
  font-size: 12px;
  .video-box {
    display: inline-block;
    vertical-align: top;
    .video {
      width: 700px;
      height: 340px;
      background-color: #000;
      border-radius: 8px;
      outline: none;
    }
  }
  .chat-box {
    display: inline-block;
    vertical-align: top;
    margin-left: 14px;
    .chat {
      background-color: #eee;
      width: 286px;
      height: 335px;
      border-radius: 8px;
      padding-top: 5px;
      margin-bottom: 3px;
      .chat-nav {
        width: 274px;
        height: 34px;
        margin: 0 auto 5px auto;
        background-color: #fff;
        border-radius: 8px 8px 0 0;
        font-size: 14px;
        .nav {
          display: inline-block;
          width: 90px;
          color: #333;
          line-height: 14px;
          border-right: 1px solid #333;
          text-align: center;
          margin-top: 10px;
        }
        .nav:nth-child(3) {
          border: none;
        }
      }
      .chose-nav {
        .content-box {
          width: 274px;
          height: 235px;
          margin: 0 auto;
          background-color: #ffffff;
          overflow: auto;
          padding-top: 12px;
          border-radius: 0 0 8px 8px;
          .content {
            line-height: 24px;
            margin: 0 15px;
            .person-name {
              color: #35a4ab;
            }
          }
        }
        .input {
          margin: 5px 6px;
          width: 200px;
        }
        p {
          width: 250px;
          margin: 0 auto;
          background-color: #ffffff;
          height: 266px;
          border-radius: 0 0 8px 8px;
          overflow: auto;
          padding: 12px;
          text-align: justify;
        }
      }
    }
  }
  .student-box {
    margin-top: 13px;
    .title {
      height: 34px;
      line-height: 34px;
      font-size: 12px;
      border: 1px solid #ccc;
      border-radius: 5px 5px 0 0;
      text-align: center;
    }
    .list-box {
      padding: 10px 20px;
      height: 200px;
      border: 1px solid #ccc;
      border-top: none;
      border-radius: 0 0 5px 5px;
      overflow: auto;
      .list {
        display: inline-block;
        margin: 5px 16px;
        cursor: pointer;
        .student-name {
          color: #ffffff;
          text-align: center;
          background-color: #25a9e1;
        }
      }
    }
  }
  .works {
    margin: 20px 0;
    .work {
      display: inline-block;
      margin-right: 5px;
      text-align: center;
    }
    .work:nth-last-child(1) {
      margin-right: 0;
    }
  }
  .evaluate {
    margin-bottom: 40px;
    .collapse-row {
      margin: 20px 30px;
      div {
        display: inline-block;
      }
      .name {
        color: #25a9e1;
        margin-right: 5px;
      }
    }
  }

  .pagination {
    text-align: center;
    margin-bottom: 20px;
    margin-top: 30px;
  }
}
.active {
  color: #25a9e1 !important;
}
.is-link {
  background-color: #ffae00 !important;
}
.el-popover {
  padding: 7px 0;
  min-width: 0;
  .btn {
    height: 25px;
    line-height: 25px;
    padding: 0 7px;
    cursor: pointer;

    .btn:hover {
      color: #25a9e1;
    }
  }
}
</style>
