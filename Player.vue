<template>
  <!-- <div :id="id" class="flowplayer" preload="true"></div> -->
  <video :width="innerWidth" :height="innerHeight" autoplay id="player">
    <source :src="video_link" type="video/mp4">
    <source :src="video_link" type="video/mpeg">
    Your browser does not support the video tag.
  </video>
</template>

<script>
export default {
  name: 'Player',
  
  props: ['video_link', 'poster', 'id'],

  data() {
    return {
      innerHeight : window.innerHeight,
      innerWidth : window.innerWidth,
      popoverlayBg: true,
      canClose: false,
      flowplayer: null,
      player: null,
    };
  },
  mounted() {
    this.player = document.getElementById("player");
    this.player.controls = false;

    // this.initialVideo();
  },
  beforeDestroy() {
    console.log('pause');
    this.player.pause();
  },

  methods: {
    // async initialVideo() {
    //   console.log('initiate');
    //   const video_link = this.video_link;
    //   var _this = this;
      // setTimeout(
      //   function (video_link, a) {
      //     var container = document.getElementById(a.id)
      //     a.flowplayer = flowplayer(container, {
      //     //  key:key,
      //       loop: true,
      //       autoplay: false,
      //       autopause: true,
      //       logo: 'https://www.mall91.com/img/mall91_icon.d49ebc27.png',
      //       title:'Video title goes here',
      //       controls: false,
      //       volume: 1,
      //       clip: {
      //         sources: [
      //           {
      //             type: 'video/webm',
      //             autoplay: true,
      //             src: video_link,
      //           }
      //         ]
      //       }
      //     })
      //     a.flowplayer.play()
      //   },50,video_link,
      //   _this
      // )


      // _this.flowplayer = await flowplayer(`#${_this.id}`, {
      //   src: video_link,
      //   autoplay: false,
      //   autopause: true,
      //   logo: 'https://www.mall91.com/img/mall91_icon.d49ebc27.png',
      //   title:'Video title goes here',
      //   type: 'video/webm',
      //   controls: false,
      //   volume: 1,
      //   // ratio:"320:640",
      //   // poster:_this.poster,
      // });
      // _this.flowplayer.on('canplay', function(e) {
      //   console.info(e.type)
      //   _this.flowplayer.play()
      // })
    // },
  },
};
</script>

<style scoped>

</style>
