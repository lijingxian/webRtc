<template>
  <div class="live-detail-page">
    <div class="live-box">
      <div class="video-box">
        <Rtc
          :screen-type="screenType"
          ref="webrtc"
          width="100%"
          :user-info="userInfo"
          :roomId="roomId"
          @connectingMike="connectingMike"
          @join="onJoinUser"
          @message="onMessage"
        ></Rtc>
      </div>
      <div class="chat-box">
        <div class="chat">
          <div class="chat-nav">
            <span class="nav" v-for="(item, index) in chatNav" :key="index" :class="{ active: navChoose == index }" @click="chooseNav(index)">{{ item }}</span>
          </div>
          <div class="chose-nav" v-show="navChoose === 0">
            <div class="content-box">
              <div class="content" v-for="(item, index) in chatList" :key="index">
                <span class="person-name">{{ item.name }}：</span>
                <span class="chat-content">{{ item.chat }}</span>
              </div>
            </div>
            <el-input class="input" v-model="chatInput" placeholder="请输入问题"></el-input>
            <el-button type="warning" size="small" @click="sendMessage">发送</el-button>
          </div>
          <div class="chose-nav" v-show="navChoose === 1">
            <p>{{ liveIntroduce }}</p>
          </div>
          <div class="chose-nav" v-show="navChoose === 2">
            <p>{{ teacherIntroduce }}</p>
          </div>
        </div>
        <el-row>
          <el-button plain="plain" size="small" @click="changeScreen('live')"
            >直播屏</el-button
          >
          <el-button plain="plain" size="small" @click="changeScreen('design')"
            >课件屏</el-button
          >
          <el-button plain="plain" size="small" @click="onJoin">加入直播</el-button>
          <el-button class="float-right" plain="plain" size="small" @click="onLeave">退出直播间</el-button>
          <div class="clear"></div>
        </el-row>
      </div>
      <div class="student-box">
        <div class="title">学生列表</div>
        <div class="list-box">
          <div class="list" slot="reference" v-for="(item, index) in studentList" :key="index">
            <img class="student-img" :src="item.imgUrl" />
            <p class="student-name" :class="{ 'is-link': item.isLink }">{{ item.name }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import pic from '@/assets/img/live/pic.png';
import Rtc from '@/components/Rtc';

export default {
  name: 'Student',
  components: {
    Rtc
  },
  data() {
    return {
      screenType: 'design',
      userInfo: {
        id: Math.ceil(Math.random() * 10),
        name: '李同学' + Math.ceil(Math.random() * 10),
        role: 'student',
        openVideo: false
      },
      disabled: false,
      disabledIds: [],
      img: null,
      roomId: 'public-room',
      liveSrc: '/static/movie/test.mp4',
      chatNav: ['在线答疑', '直播简介', '教师简介'],
      navChoose: 0,
      chatList: [
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' },
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' },
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' },
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' },
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' },
        { name: '谁谁谁', chat: '问题内容', isTeacher: 'false' }
      ],
      liveIntroduce: '直播简介直播简介直播简介直播简介直播简介直播简介直播简介直播简介直播简介直播简介',
      teacherIntroduce:
        '老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介老师简介',
      chatInput: '',
      studentList: [],
      id: '',
      works: [
        { pic: pic, name: '优秀作品', url: '' },
        { pic: pic, name: '优秀作品', url: '' },
        { pic: pic, name: '优秀作品', url: '' },
        { pic: pic, name: '优秀作品', url: '' },
        { pic: pic, name: '优秀作品', url: '' },
        { pic: pic, name: '优秀作品', url: '' }
      ],
      evaluateList: [
        { name: '王小明', text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！' },
        { name: '王小明', text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！' },
        { name: '王小明', text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！' },
        { name: '王小明', text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！' },
        { name: '王小明', text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！' },
        { name: '王小明', text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！' },
        { name: '王小明', text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！' },
        { name: '王小明', text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！' },
        { name: '王小明', text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！' },
        { name: '王小明', text: '李老师您好，您所申请的“java基础课程”已通过审核，即时开课！' }
      ],
      total: 100,
      currentPage: 1
    };
  },
  computed: {},
  watch: {},
  mounted() {
    setTimeout(() => {
      this.screenType = 'live';
    }, 100);
  },
  created() {},
  methods: {
    // 查看视频在线所有人员信息
    onJoinUser(userList) {
      console.log(userList);
      this.studentList = [];
      for (const user of userList) {
        this.studentList.push({
          id: user.userId,
          name: user.name
        });
      }
    },
    onJoin() {
      this.$refs.webrtc.join(this.userInfo);
      this.id = Math.ceil(Math.random() * 10);
    },
    onLeave() {
      this.$refs.webrtc.leave();
      this.studentList.map(item => {
        if (item.id === this.userInfo.id) {
          this.studentList.splice(item, 1);
        }
      });
    },
    sendMessage() {
      // 如果该学生被禁言，不能提问
      if (this.disabledIds.includes(this.userInfo.id)) {
        return;
      }
      let message = {
        from: {
          role: 'student',
          id: this.userInfo.id,
          name: this.userInfo.name
        },
        chat_type: 'group',
        payload: {
          body: {
            msg: this.chatInput, // 消息内容
            type: 'txt' // 文本消息类型
          }
        }
      };
      // 发送消息
      this.$refs.webrtc.sendMessage(message);
      this.chatList.push({
        name: this.userInfo.name,
        id: this.userInfo.id,
        isTeacher: false,
        chat: this.chatInput
      });
      this.chatInput = '';
    },
    onMessage(message, type) {
      console.log(message);
      if (type === 'message' && message) {
        if (message.chat_type && message.chat_type === 'disabled' && message.payload.body.id === this.userInfo.id) {
          this.disabled = message.payload.body.disabled;
          this.disabledIds.push(this.userInfo.id);
          return;
        }
        this.chatList.push({
          name: message.from.name,
          isTeacher: message.from.role === 'teacher',
          chat: message.payload.body.msg
        });
      } else if (type === 'connectingMike') {
        this.connectingMike(message);
      } else if (type === 'disConnectingMike') {
        this.disConnectingMike(message);
      } else if (type === 'changeScreen') {
        this.changeScreen(message);
      }
    },
    changeScreen(type) {
      console.log(type);
      this.screenType = type;
      this.$refs.webrtc.changeScreen(type, true);
    },
    connectingMike(message) {
      console.log('connectingMike');
      this.$refs.webrtc.connectingMike(message, false);
    },
    disConnectingMike(message) {
      console.log('disConnectingMike');
      this.$refs.webrtc.disConnectingMike(message, false);
    },
    // 以下函数自己修改
    onCapture() {
      this.img = this.$refs.webrtc.capture();
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
    }
  }
};
</script>
<style lang="scss">
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
