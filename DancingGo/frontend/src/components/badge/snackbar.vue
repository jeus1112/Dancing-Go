<template>
    <v-snackbar
      v-model="snackbar"
      :vertical="vertical"
      :height="height"
      :width="width"
      :timeout="timeout"
      centered
      content-class="badgeSnackbar"
      color="rgba(43, 29, 59, 0.8)"
    >
      <span v-if="this.$store.getters.langMode=='한국어'" class="badgeText mt-5 mb-2">🎉축하합니다!🎉</span>
      <span v-else class="badgeText mt-5 mb-2">🎉Congratulations!🎉</span>
      <span v-if="this.$store.getters.langMode=='한국어'" class="badgeText mb-5">뱃지를 획득했습니다!</span>
      <span v-else class="badgeText mb-5">You've got a badge!</span>

      <img class="badgeImg mb-2" style="opacity:1;" :src="badgeImg">
      <span v-if="this.$store.getters.langMode=='한국어'" class="badgeName mt-4">{{ badgeNameKor }}</span>
      <span v-else class="badgeName mt-4">{{ badgeNameEng }}</span>
      <span v-if="this.$store.getters.langMode=='한국어'" class="badgeCondition mt-2">{{ badgeConditionKor }}</span>
      <span v-else class="badgeCondition mt-2">{{ badgeConditionEng }}</span>
      <template v-slot:action="{ attrs }">
        <v-btn
          color="indigo"
          text
          v-bind="attrs"
          @click="snackbar = false"
        >
        </v-btn>
      </template>
    </v-snackbar> 
</template>
<script>
import { mapGetters } from 'vuex'
import http from '@/http.js'
export default {
  props:{
    gameover: {
      type: Number,
    },
    totalScore: {
      type: Number,
    }
  },
  data () {
    return{
      snackbar: false,
      userInfo: {},
      badgeInfo: [],
      userNickname:'',
      badgeNum: '',
      badgeNameKor: '',
      badgeNameEng: '',
      badgeConditionKor: '',
      badgeConditionEng: '',
      badgeImg: '',
      badgelist:[],
      //--- snackbar 부분 ---
      
      vertical: true,
      height: '360px',
      width: '450px',
      timeout : 1000,
      tmpSnackbar: false,
      name: '시작이 반이다',
      condition: '첫 곡을 완료하세요',
      cnt: 0
    }
  },
  computed:{
    ...mapGetters(['token', 'user']),
  },
  watch: {
    cnt (val) {
      if (val === this.badgelist.length) {
        this.goEmit()
      }
    }
  },
  mounted() {
    http.get("/user/gameinfo/" + this.user.userId)
    .then((res)=>{
      this.userInfo = res.data
      this.userNickname = res.data.userNickname

      this.checkBadge();
    })
  },
  methods:{
    // checkBadge() > getBadge(addChellenge(), addBadgeList()) > showBadge()
    checkBadge(){
      http.get("/challenge/onlyMyBadge/"+ this.userNickname)
      .then((res)=>{
        this.badgeInfo = res.data
      })

      const totalPlayCnt = this.userInfo.totalPlayCnt //mounted

      // 획득 조건 판단 후 badgelist에 {bNumber:번호(Number), bNameKor:한글뱃지이름(String), bNameEng:영어뱃지이름(String)} push
      setTimeout(function(){
        this.getBadge(totalPlayCnt)
      }.bind(this),100)

      // badgelist에 담긴 획득한 뱃지를 순차적으로 보여줌
      let i =0;
      var sT = setTimeout(function run(){
        if(i==this.badgelist.length){
          this.$emit('snackbarfinished')
          clearTimeout(sT)
        }else{
          this.showBadge(this.badgelist[i++])
          this.cnt ++
          setTimeout(run.bind(this),2000);
        }
      }.bind(this),2000)
    },
    goEmit () {
      this.$emit('snackbarfinished')
    },
    showBadge(badgelist){
        this.badgeNum = badgelist.bNumber
        this.badgeNameKor = badgelist.bNameKor
        this.badgeNameEng = badgelist.bNameEng
        this.badgeConditionKor = badgelist.bConditionKor
        this.badgeConditionEng = badgelist.bConditionEng
        this.badgeImg = '/images/badge/'+badgelist.bNumber+'.png'
        this.snackbar = true
    },

    addChallenge(challengeAddReq){
      http.post("/challenge/addChallenge", challengeAddReq)
    },

    addBadgeList(badgeNum, badgeNameKor, badgeNameEng, badgeConditionKor, badgeConditionEng){
      this.badgelist.push({bNumber:badgeNum, bNameKor:badgeNameKor, bNameEng:badgeNameEng, bConditionKor:badgeConditionKor, bConditionEng: badgeConditionEng})
    },

    getBadge(totalPlayCnt){
      http.get("/user/gameinfo/" + this.user.userId)
      .then((res)=>{
        // 1001, 첫 완곡
        if(this.gameover!=1 && totalPlayCnt==1 && !this.badgeInfo.includes(1001)){
          const challengeAddReq = {
            badgeId : 1001,
            userNickname : this.userInfo.userNickname,
          }
          this.addChallenge(challengeAddReq)
          this.addBadgeList(1001,"시작이 반이다","Well begun is half done","첫 곡을 완료하세요!","finish your first dance")
        }
        // 1002, 첫 게임오버
        if(this.gameover==1 && !this.badgeInfo.includes(1002)){
          const challengeAddReq = {
            badgeId : 1002,
            userNickname : this.userInfo.userNickname,
          }
          this.addChallenge(challengeAddReq)
          this.addBadgeList(1002,"실패는 성공의 어머니","Failure is but a stepping stone to success","첫 완곡 실패... ","first fail to finish the dance")
        }
        // 1003, top 100
        if(this.gameover!=1 && res.data.rank <= 100 && !this.badgeInfo.includes(1003)){
          const challengeAddReq = {
            badgeId : 1003,
            userNickname : this.userInfo.userNickname,
          }
          this.addChallenge(challengeAddReq)
          this.addBadgeList(1003,"Top 100!","Top 100!","100등 안에 들어보세요!","In the top 100")
        }
        // 1004, top 10
        if(this.gameover!=1 && res.data.rank <= 10 && !this.badgeInfo.includes(1004)){
          const challengeAddReq = {
            badgeId : 1004,
            userNickname : this.userInfo.userNickname,
          }
          this.addChallenge(challengeAddReq)
          this.addBadgeList(1004,"Top 10!","Top 10!","10등 안에 들어보세요!","in the top 10")
        }
        // 1005, top 3
        if(this.gameover!=1 && res.data.rank <= 3 && !this.badgeInfo.includes(1005)){
          const challengeAddReq = {
            badgeId : 1005,
            userNickname : this.userInfo.userNickname,
          }
          this.addChallenge(challengeAddReq)
          this.addBadgeList(1005,"단상에 내 이름이!","Take podium","3등 안에 들어보세요!","ranked 3rd")
        }
        //1006, top 2
        if(this.gameover!=1 && res.data.rank <= 2 && !this.badgeInfo.includes(1006)){
          const challengeAddReq = {
            badgeId : 1006,
            userNickname : this.userInfo.userNickname,
          }
          this.addChallenge(challengeAddReq)
          this.addBadgeList(1006,"앞으로 한명!","One to go!","2등안에 들어보세요!","ranked 2nd")
        }
        // 1007, top 1
        if(this.gameover!=1 && res.data.rank == 1 && !this.badgeInfo.includes(1007)){
          const challengeAddReq = {
            badgeId : 1007,
            userNickname : this.userInfo.userNickname,
          }
          this.addChallenge(challengeAddReq)
          this.addBadgeList(1007,"내가 바로 춤.신.춤.왕.","I am the king of dance!!","우와! 1등입니다!","ranked 1st")
        }
        // 1008, 점수가 10점이하
        if(this.gameover!=1 && this.totalScore <= 10 && !this.badgeInfo.includes(1008)){
          const challengeAddReq = {
            badgeId : 1008,
            userNickname : this.userInfo.userNickname,
          }
          this.addChallenge(challengeAddReq)
          this.addBadgeList(1008,"춤이 추고 싶어?","Do you want to dance?","10점 이하를 기록했어요", "score lower than 10")
        }
        // 1009, 점수가 100점
        if(this.gameover!=1 && this.totalScore == 100 && !this.badgeInfo.includes(1009)){
          const challengeAddReq = {
            badgeId : 1009,
            userNickname : this.userInfo.userNickname,
          }
          this.addChallenge(challengeAddReq)
          this.addBadgeList(1009,"최고의 댄서이시군요!","You are the best dancer!","100점을 기록했어요!","score 100")
        }
      })
    }
  }
}
</script>

<style>

#check {
  position: absolute;
  top: 100px;
  left: 720px;
  color: red;
}

#background {
  position: absolute;
  width: 100vw;
  height: 100vh;
}

.badgeSnackbar {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 360px;
  width: 450px;
}

.badgeImg {
  width: 160px;
}

.badgeText {
  font-size: 20px;
}

.badgeName {
  font-size: 20px;
}

.badgeCondition {
  font-size: 15px;
}

</style>
