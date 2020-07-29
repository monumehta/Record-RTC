<template>
  <div class="">
    <div class="video-screen">
      <div class="swiper-container">
        <div class="swiper-wrapper" v-if="music.length>0">
          <div class="swiper-slide" v-for="(item, i) in music" :key="i">
            <Player v-if="currentSlide == i"
              class="vdo-screen"
              :video_link = item.url
              :id="`player${i}`"
              poster="http://gci.ph/my/wp-content/themes/blessing/images/video_poster2.jpg"
            />
            <div class="image" v-else>
                <img class="imagesrc" src="../assets/logo.webp">
            </div>
          </div>
        </div>
      </div>
    </div>
    <button class="back" @click="$router.push('record')">New</button>
    <loader91 v-if="isLoading" />
  </div>
</template>

<script>
import Player from './Player';

export default {
  name: 'home',

  components: {
    Player,
  },

  data () {
      return {
          innerHeight : window.innerHeight,
          innerWidth : window.innerWidth,
          isLoading:false,
          xauthtoken:this.$cookies.get("X-Auth-Token"),
          music:[],
          currentSlide :0,
      }
  },

  computed: {
      user: {
        get() {
            return this.$store.state.user;
        },
        set(newValue) {
            return this.$store.dispatch("setUser", newValue);
        }
        }
    },

    async mounted() {
        try {
        __native_money91.checkForPermissionResult = function(res) {
            if (res === '') {
                console.log('Permission success')
            } else {
                console.log('Error in permission')
            }
        };

        var _ = [
            'android.permission.CAMERA',
            'android.permission.RECORD_AUDIO',
            'android.permission.WRITE_EXTERNAL_STORAGE',
            'android.permission.READ_EXTERNAL_STORAGE',
        ];
        JSBridgePlugin.checkForPermission(
            _,
            true,
            '__native_money91.checkForPermissionResult'
        );
        } catch (e) {
        console.log('JSBridgePlugin not defined',e)
        }

        this.isLoading = true;
        // console.log(this.xauthtoken)

        if (this.xauthtoken !== null) {

            if (Object.keys(this.user).length == 0) {
                let userDetails = await this.axios.post(
                `${window.serverUrl}api/createuser`,
                {
                    xauthtoken: this.xauthtoken
                });

                if (userDetails !== null && userDetails.data.status) {
                    this.user = userDetails.data.user;
                    this.isLoading = false;
                    
                } else {
                    this.$toast.bottom(userDetails.data.message);
                }
            } else {
                this.isLoading = false;
            }

            let vdo = await this.axios.post(
            `${window.serverUrl}api/video`,
            {
                xauthtoken: this.xauthtoken
            });

            if (vdo !== null && vdo.data.status) {
                this.music = vdo.data.data;
                setTimeout((t)=> {
                    var swiper = new Swiper('.swiper-container', {
                        direction: 'vertical',
                        slidesPerView: 1,
                        spaceBetween: 0,
                        mousewheel: true,
                    });
                    
                    swiper.on('slideChange', function () {
                        console.log('slide changed');
                        console.log(swiper.realIndex)
                        t.currentSlide = swiper.realIndex

                    });

                },500,this)
                
                
            } else {
                this.$toast.bottom(vdo.data.message);
            }
            
        }

    
    }

}
</script>

<style scoped>

</style>
