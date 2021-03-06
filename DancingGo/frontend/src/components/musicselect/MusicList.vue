
<template>
  <div>
    <div id="now">
      <div id="difficulty">
        <span v-if="musics[activeIndex].difficulty===1">★</span>
        <span v-else-if="musics[activeIndex].difficulty===2">★★</span>
        <span v-else-if="musics[activeIndex].difficulty===3">★★★</span>
      </div>
      <img id="now-image" @click="openModal" :src="'images/musicselect/'+musics[activeIndex].fileName+'.png'" alt="">
      <div v-if="mode==='Practice'" id="active-music-info" class="text-center my-3">
        <p v-if="this.$store.getters.langMode=='한국어'">{{ musics[activeIndex].songNameKor }}</p>
        <p v-else>{{ musics[activeIndex].songNameEng }}</p>
        <p v-if="this.$store.getters.langMode=='한국어'">{{ musics[activeIndex].singerKor }}</p>
        <p v-else>{{ musics[activeIndex].singerEng }}</p>
        <p v-if="this.$store.getters.langMode=='한국어'">플레이 횟수: {{ myPlayCnt }}</p>
        <p v-else>times played</p>
      </div>
      <div v-else id="active-music-info" class="text-center my-5">
        <p v-if="this.$store.getters.langMode=='한국어'">
          {{ musics[activeIndex].songNameKor }} - {{ musics[activeIndex].singerKor }}</p>
        <p v-else>{{ musics[activeIndex].songNameEng }} - {{ musics[activeIndex].singerEng }}</p>
        <p v-if="this.$store.getters.langMode=='한국어'">최고 점수: {{ musics[activeIndex].value || 0 }}점</p>
        <p v-else>Highest score: {{ musics[activeIndex].value ||0 }} points</p>
        <v-btn
          v-if="this.$store.getters.langMode=='한국어'"
          @click="openRink"
          small
          id="button"
          style="opacity: 80%;">
          랭킹 확인 🏆
        </v-btn>
        <v-btn
          v-else
          @click="openRink"
          small
          id="button"
          style="opacity: 80%;">
          View Ranks 🏆
        </v-btn>
      </div>
    </div>
    <div class="example-3d">
      <swiper class="swiper" ref="musiclist" @slide-change-transition-end="getActive" :options="swiperOptionThumbs" >
        <Music
          class="align-self-center"
          v-for="(music, idx) in musics"
          :key="idx"
          :music="music"
        />
      </swiper>
    </div>

    <v-dialog
      v-model="isRankOpen"
      max-width="500px"
      > <SongRank 
      :songId="musics[activeIndex].songId"
      @closeRank="isRankOpen" class="mypageModal"/>
    </v-dialog>
    <v-dialog
      v-model="isModalOpen"
      max-width="350px">
      <Modal
        v-if="this.$store.getters.langMode=='한국어'"
        :modalTitle="'알림'"
        :modalContent="'댄스 한 판 즐겨볼까요?💃'"
        :buttonO="'시작'"
        :buttonX="'취소'"
        @clickO="goToGame(musics[activeIndex].songId)"
        @clickX="closeModal"
      />
      <Modal
        v-else
        :modalTitle="'Notice'"
        :modalContent="'Let\'s enjoy a dance!💃'"
        :buttonO="'START'"
        :buttonX="'CANCEL'"
        @clickO="goToGame(musics[activeIndex].songId)"
        @clickX="closeModal"
      />
    </v-dialog>
    <!-- 선택 시 효과음 -->
    <audio src="sounds/select.mp3" ref="selecteffect"></audio>
    <!-- 선택한 곡 -->
    <audio :src="'songs/'+musics[0].fileName+'.mp3'" ref="backgroundSound" autoplay></audio>
  </div>
</template>

<script>
  import { Swiper } from 'vue-awesome-swiper'
  import Music from '@/components/musicselect/Music.vue'
  import Modal from '@/components/Modal.vue'
  import 'swiper/css/swiper.css'
  import { mapState, mapGetters } from 'vuex'
  import SongRank from '@/components/musicselect/songRank.vue'
  import http from '@/http.js';

  export default {
    name: 'MusicList',
    components: {
      Swiper,
      Music,
      SongRank,
      Modal,
    },
    props: {
      mode: {
        type: String
      }
    },
    computed: {
      ...mapGetters(['token', 'user']),
      ...mapState('music', {
        musics: state => state.musics
      }),
      activeIndex () {
        return this.activeIdx
      },
      changeBg () {
        return this.$store.getters.backgroundVolume*(0.01)
      },
      changeEffect () {
      return this.$store.getters.effectVolume*(0.01)
      },
    },
    watch: {
      changeBg (val) {
      this.$refs.backgroundSound.volume = val
      },
      changeEffect (val) {
      this.$refs.selecteffect.volume = val
      },
      activeIdx : function() {
        const body = {
          songId: this.musics[this.activeIndex].songId,
          userNickname: this.user.userNickname
        }
        http
          .post('/score/findMyScore/', body)
          .then((res) => {
            this.myPlayCnt = res.data.playCnt
          })
      }
    },
    data() {
      return {
        isModalOpen: false,
        activeIdx: 0,
        swiperOptionTop: {
          spaceBetween: 10
        },
        swiperOptionThumbs: {
          effect: 'coverflow',
          grabCursor: true,
          centeredSlides: true,
          slidesPerView: 'auto',
          loopAdditionalSlides: 3,
          coverflowEffect: {
            rotate: 0,
            stretch: 0,
            depth: 100,
            modifier: 1,
            slideShadows : true
          },
          keyboard: {
            enabled: true
          },
          slideToClickedSlide: true
        },
        isRankOpen: false,
        myPlayCnt: 0
      }
    },
    methods: {
      getActive () {
        this.$refs.selecteffect.play()
        const swiper = this.$refs.musiclist.$swiper
        this.activeIdx = swiper.activeIndex
        this.$refs.backgroundSound.src = 'songs/'+this.musics[this.activeIndex].fileName+'.mp3'
        this.$refs.backgroundSound.play()
      },
      goToPractice (songId) {
        this.$router.push({ name: 'Practice', params: { songId: songId } })
      },
      songRank (songId) {
        alert(songId);
      },
      openRink () {
        this.$refs.selecteffect.play()
        this.isRankOpen = true
      },
      closeRink() {
        this.isRankOpen = false
      },
      openModal () {
        this.$refs.selecteffect.play()
        this.isModalOpen = true
      },
      closeModal () {
        this.$refs.selecteffect.play()
        this.isModalOpen = false
      },
      goToGame (songId) {
        this.$refs.selecteffect.play()
        setTimeout(function() {
          this.$router.push({ name: this.$route.query.mode, params: { songId: songId } })
        }.bind(this), 500)
      }
    },
    mounted () {
      this.$refs.backgroundSound.volume = this.$store.getters.backgroundVolume*(0.01)
      this.$refs.selecteffect.volume = this.$store.getters.effectVolume*(0.01)
      this.$refs.backgroundSound.play()
      const body = {
        songId: this.musics[this.activeIndex].songId,
        userNickname: this.user.userNickname
      }
      http
        .post('/score/findMyScore/', body)
        .then((res) => {
          this.myPlayCnt = res.data.playCnt
        })
    }
  }
</script>

<style lang="scss" scoped>
  #button {
  color: white;
  background-color: black;
  box-shadow: 0 0 10px white;
  }

  #now {
    background-color: rgba( 0, 0, 0, 0.8 );
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -70%);
    box-shadow: 0 0 20px white;
    border-radius: 2%;

    img {
      width: 250px;
      height: 250px;
      border-radius: 2%;
      cursor: pointer;
    }
  }

  #difficulty {
    text-shadow: 0 0 4px yellow;
    color: white;
    -webkit-text-stroke: 0.3px purple;
    font-size: 30pt;
    position: absolute;
    transform: translate(-50%, -50%);
    left: 50%;
  }

  #active-music-info {
    line-height: 70%;
    text-shadow: 0 0 4px purple;
    color: white;
    -webkit-text-stroke: 0.3px purple;
  }

  .example-3d {
    width: 100%;
    height: 250px;
    position: absolute;
    bottom: 0vh;
  }
  .swiper-wrapper {
    width: 100%;
    height: 250px;
    position: absolute;
    bottom: 0vh;
    display: flex;
    object-fit: visible;
  }
  .swiper {
    height: 100%;
    width: 100%;
    vertical-align: middle;

    .swiper-slide {
      display: flex;
      justify-content: center;
      width: 180px;
      height: 180px;
      text-align: center;
      font-weight: bold;
      background-color: #2C8DFB;
      background-position: center;
      background-size: cover;
      border-radius: 2%;
    }
  }

  #rankIcon {
    object-fit: contain;
    width: 8vh;
    height: 8vh;
    cursor: pointer;
  }
</style>