<script>
import {
  CheckSquareTwoTone,
  ClockCircleTwoTone,
  DollarOutlined,
  GiftTwoTone,
  StarTwoTone,
} from "@ant-design/icons-vue";
import { message } from "ant-design-vue";
import { cancelFollowing, follow } from "../request/friend";
import { getProfileData2 } from '../request/profile';
import {
  browseinAPI,
  getLatest,
  getOnline,
  getRecommendUsers,
  signinAPI,
} from "../request/request";
import store from "../store/store";
import { dateFormat, moment } from "../utils";
export default {
  name: "rightBar",
  data() {
    return {
      recommendIsLoading: true,
      latestIsLoading: true,
      onlineIsLoading: true,
      recommendUsers: [],
      latest: [],
      online: [],
      inTime: new Date(),
      personality:null,
      roleId:[],
    };
  },
  components: {
    ClockCircleTwoTone,
    StarTwoTone,
    GiftTwoTone,
    CheckSquareTwoTone,
    DollarOutlined,
  },
  methods: {
    async ChangeId(roleId) {
      try {
        //console.log(this.user.roleId)
        const res = await getProfileData2(roleId);
        // 在这里处理response，例如解析JSON数据  
        console.log(res.data.data);
        return res.data.data;
        //console.log(this.personality); // 打印从服务器获取的数据  
      } catch (error) {
        console.error('Error fetching profile data:', error);
      }
    },
    gotoProfileHandler(userId) {
      this.$router.push(`/profile/${userId}`).then(() => {
        window.scrollTo(0, 0);
      });
    },
    followHandler(user) {
      follow(user.userId).then(() => {
        user.isFollowed = true;
      });
    },
    cancelFollowHandler(user) {
      cancelFollowing(user.userId).then(() => {
        user.isFollowed = false;
      });
    },
    async signin() {
      try {
        const res = await signinAPI(store.state.currentUser.userId);
        console.log(res);
      } catch (error) {
        message.info(error.response.data);
      }
    },
    async timeGet() {
      try {
        if (new Date() - this.inTime > 1000 * 60 * 10) {
          const res = await browseinAPI();
        } else {
          message.info("登录未满10min");
        }
      } catch (error) {
        message.info(error.response.data);
      }
    },
  },
   async mounted() {
    await getRecommendUsers().then((res) => {
      this.recommendUsers = res.data.data;
      console.log(this.recommendUsers);
      this.personality = new FormData()
      this.recommendUsers.forEach(async item =>{
        //console.log(item.roleId)
        if(item.roleId == 0){
          this.personality.set(item.roleId, " ")
          //console.log(this.personality.get(item.roleId));
        }
        else{
         this.personality.set(item.roleId, await this.ChangeId(item.roleId))}
         //console.log(this.personality.get(item.roleId));
      })
      this.recommendIsLoading = false;
    });

    getLatest().then((res) => {
      this.latest = res.data.data;

      this.latest.forEach((item) => {
        item.time = moment(item.time, dateFormat).fromNow();
      });

      this.latestIsLoading = false;
    });

    getOnline().then((res) => {
      this.online = res.data.data;
      this.onlineIsLoading = false;
    });
  },
};
</script>
<template>
  <div className="right-bar">
    <div className="container">
      <div class="item" v-if="recommendIsLoading">
        <a-spin />
      </div>
      <div className="item" v-else>
        <span class="gray">向你推荐</span>
        <div
          className="user recommend"
          v-for="user in recommendUsers"
          :key="user.userId"
        >
          <div className="user-info">
            <img :src="user.profilePic" />
            <span class="name" @click="() => gotoProfileHandler(user.userId)">{{
              user.nickname
            }}</span>
            
            <span class="personality" >{{ this.personality.get(user.roleId) }}</span>
            <span class="location">{{ user.location }}</span>
            <span class="language">{{ user.language }}</span>
            <p class="reason">
              {{ user.reason }}
            </p>
          </div>
          <div className="btn">
            <a-button
              v-if="user.isFollowed"
              @click="() => cancelFollowHandler(user)"
            >
              已关注
            </a-button>
            <a-button v-else type="primary" @click="() => followHandler(user)">
              关注
            </a-button>
          </div>
        </div>
      </div>
      <div class="item" v-if="latestIsLoading">
        <a-spin />
      </div>
      <div v-else className="item">
        <span>最新动态</span>
        <div className="user" v-for="item in latest" :key="item.time">
          <div className="user-info">
            <img :src="item.profilePic" />
            <div class="body">
              <p class="name" @click="() => gotoProfileHandler(item.userId)">
                {{ item.nickname }}
              </p>
              <p class="content">{{ item.content }}</p>
            </div>
          </div>
          <span class="date">
            {{ item.time }}
          </span>
        </div>
      </div>
      <div class="item" v-if="onlineIsLoading">
        <a-spin />
      </div>
      <div v-else className="item">
        <span>在线好友</span>
        <div className="user" v-for="user in online" :key="user.userId">
          <div className="user-info">
            <img :src="user.profilePic" />
            <div className="online"></div>
            <span class="name" @click="() => gotoProfileHandler(user.userId)">
              {{ user.nickname }}
            </span>
          </div>
        </div>
      </div>
      <div className="item">
        <div className="header">
          <span>每日任务</span>
        </div>
        <div className="score_item">
          <div className="pic"><ClockCircleTwoTone /></div>
          <div className="content">
            <span className="title">签到</span>
            <span className="des"
              ><DollarOutlined style="color: goldenrod" />40积分</span
            >
          </div>
          <a-button @click="signin">获取积分</a-button>
        </div>
        <div className="score_item">
          <div className="pic"><StarTwoTone /></div>
          <div className="content">
            <span className="title">浏览10分钟</span>
            <span className="des"
              ><DollarOutlined style="color: goldenrod" />5积分</span
            >
          </div>
          <a-button @click="timeGet">获取积分</a-button>
        </div>
        <div className="score_item">
          <div className="pic"><GiftTwoTone /></div>
          <div className="content">
            <span className="title">分享</span>
            <span className="des"
              ><DollarOutlined style="color: goldenrod" />5积分</span
            >
          </div>
          <a-button>获取积分</a-button>
        </div>
        <div className="score_item">
          <div className="pic"><CheckSquareTwoTone></CheckSquareTwoTone></div>
          <div className="content">
            <span className="title">签到</span>
            <span className="des"
              ><DollarOutlined style="color: goldenrod" />5积分</span
            >
          </div>
          <a-button>获取积分</a-button>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
@import "../style.scss";

.right-bar {
  .score_item {
    padding: 10px;
    display: flex;
    .pic {
      width: 50px;
      height: 50px;
      font-size: 30px;
    }
    .content {
      flex: 1;
      display: flex;
      flex-direction: column;
      gap: 2px;
      .title {
        font-size: 16px;
        font-weight: 500;
      }
    }
  }
  @include themify($themes) {
    flex: 3;
    position: sticky;
    top: 50px;
    height: calc(100vh - 50px);
    overflow: scroll;
    background-color: themed("bgSoft");

    &::-webkit-scrollbar {
      display: none;
    }

    @include mobile {
      display: none;
    }

    @include tablet {
      display: none;
    }

    .gray {
      color: gray;
    }

    .container {
      padding: 20px;

      .item {
        -webkit-box-shadow: -1px 5px 19px 0px rgba(194, 188, 194, 0.31);
        -moz-box-shadow: -1px 5px 19px 0px rgba(194, 188, 194, 0.31);
        box-shadow: -1px 5px 19px 0px rgba(194, 188, 194, 0.31);
        padding: 20px;
        margin-bottom: 20px;
        background-color: themed("bg");

        .user {
          display: flex;
          align-items: center;
          justify-content: space-between;
          margin: 10px 0;

          &.recommend {
            margin-top: 40px;
          }

          .user-info {
            display: flex;
            align-items: center;
            gap: 20px;
            position: relative;

            .reason {
              position: absolute;
              top: -20px;
              left: 30px;
              color: #999;
              font-size: 12px;
            }

            img {
              height: 40px;
              width: 40px;
              border-radius: 50%;
              object-fit: cover;
            }

            .online {
              height: 12px;
              width: 12px;
              border-radius: 50%;
              background-color: limegreen;
              position: absolute;
              top: 0;
              left: 30px;
            }

            p {
              color: themed("textColorSoft");
            }

            .body {
              display: flex;
              flex-direction: column;
              justify-content: center;
              width: 200px;
              padding-top: 10px;

              .content {
                color: #666;
                white-space: nowrap;
                overflow: hidden;
                text-overflow: ellipsis;
              }
            }

            .name {
              font-weight: 500;
              color: themed("textColor");
              cursor: pointer;
            }

            .location,
            .language,
            .personality {
              color: #999;
              font-size: 13px;
            }
          }
        }
      }

      .btn {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 10px;
        color: #fff;
      }
    }
  }
}
</style>
