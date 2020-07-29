<template>
  <div>
      
    <div class="popup" v-if="showPopup">
        <div class="close" @click="showPopup=false"> X</div>
        <div class="song" v-for="(item,i) in song" :key="i" @click="selectAudio(item)"> {{item.name}}</div>
        <div class="song" @click="selectAudioFromPc()">Select form folder</div>
    </div>

    <audio crossOrigin="anonymous" controls style="display:none;"></audio>
    <video autoplay playsinline :width="width" :height="height"></video>

    <button class="addmusic" @click="showMusic" v-if="showRecord">Add Music</button>
    <button class="record" @click="startRecord" v-if="showRecord">Record</button>
    <button class="record" @click="stopRecord" v-if="showStop">Stop</button>
    <button class="record" @click="playRecord" v-if="showPlay">Play</button>
    <button class="next" v-if="showNext" @click="post()">Post</button>
    <button class="back" v-if="showBack" @click="backRecord()">Back</button>
    
    <loader91 v-if="isLoading" />
  </div>
    
</template>

<script>
    export default {
        data() {
            return {
                song:[
                    {
                        url:"https://money91kxtvk2qmhk.cdn.e2enetworks.net/c04bbf42-cad5-4606-bd62-0fea1e8abcf3.mpga",
                        name:'Lehanga Jass Manak (Official Song)'
                    },
                    {
                        url:"https://money91kxtvk2qmhk.cdn.e2enetworks.net/cbf41607-f5b9-46b3-857d-4e7c50347ecf.mpga",
                        name:'Badshah - DJ Waley Babu '
                    },
                    {
                        url:"https://money91kxtvk2qmhk.cdn.e2enetworks.net/7014c6ab-4f8f-4b6a-9f97-a56b9c1df9d3.mpga",
                        name:'Genda Phool song from delhi 6'
                    },
                    {
                        url:"https://money91kxtvk2qmhk.cdn.e2enetworks.net/c2b37ba6-aa9e-4af3-abb6-044d60bae375.mpga",
                        name:'लॉलीपॉप लागेलू - Pawan Singh'
                    }

                ],
                xauthtoken : this.$cookies.get("X-Auth-Token"),
                jwtToken : this.$cookies.get("JWT-TOKEN"),
                cityName : this.$cookies.get("cityName"),
                xauthid : this.$cookies.get("X-Auth-Id"),
                showPopup:false,
                showStop:false,
                showRecord:true,
                showPlay:false,
                showBack:false,
                selectedAudio:null,
                width: window.innerWidth,
                height: window.innerHeight,
                
                player: null,
                video : null,
                audioPlayer: null,
                dataAvailable:false,
                showNext:false,
                isLoading: false,
                recorder:null,
                recordedData:null,

                videoStream:null,
                audioStream:null,

            };
        },

        computed: {
            music: {
                get() {
                    return this.$store.state.music;
                },
                set(newValue) {
                    return this.$store.dispatch("setMusic", newValue);
                }
            },
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
            if(Object.keys(this.user).length ==0) this.$router.go(-1)
            await this.initiateCamera()

            
        },

        methods: {
            async initiateCamera() {
                var _this = this
                _this.isLoading = true;
                _this.video = document.querySelector('video')
                    await this.captureCamera(async function(camera) {
                    _this.video.muted = true;
                    _this.video.volume = 0;
                    _this.video.srcObject = camera;
                    _this.videoStream = camera
                    _this.isLoading = false;
                });
            },

            captureCamera(callback) {
                navigator.mediaDevices.getUserMedia({ audio: {echoCancellation:true}, video: {
                    width: this.width,
                    height: this.height
                } }).then(function(camera) {
                    callback(camera);
                }).catch(function(error) {
                    console.error('Unable to capture your camera. Please check console logs.');
                    console.error(error);
                });
            },

            getMp3Stream(callback) {
                var selector = new FileSelector();
                selector.accept = '*.mp3';
                selector.selectSingleFile(function(mp3File) {
                    try {
                        window.AudioContext = window.AudioContext || window.webkitAudioContext;
                        var context = new AudioContext();
                        var gainNode = context.createGain();
                        gainNode.connect(context.destination);
                        gainNode.gain.value = 0; // don't play for self

                        var reader = new FileReader();
                        reader.onload = (function(e) {
                            // Import callback function
                            // provides PCM audio data decoded as an audio buffer
                            context.decodeAudioData(e.target.result, createSoundSource, (e)=>{
                                console.log('File type not valid',e)
                            });
                        });
                        reader.readAsArrayBuffer(mp3File);

                        function createSoundSource(buffer) {
                            var soundSource = context.createBufferSource();
                            soundSource.buffer = buffer;
                            soundSource.start(0, 0 / 1000);
                            soundSource.connect(gainNode);
                            var destination = context.createMediaStreamDestination();
                            soundSource.connect(destination);

                            // durtion=second*1000 (milliseconds)
                            callback(destination.stream, buffer.duration * 1000);
                        }
                    } catch (error) {
                        console.log('error',error)
                    }
                    

                }, function() {
                    console.log('Please select mp3 file.');
                });
            },
            
            async selectAudioFromPc() {
                var _this = this
                await this.getMp3Stream(async function(mp3Stream) {

                    navigator.mediaDevices.getUserMedia({
                        video: {
                            width: _this.width,
                            height: _this.height
                        }
                    }).then(async function(camera) {
                        _this.video.muted = true;
                        _this.video.volume = 0;                        
                        _this.video.srcObject = camera;
                        _this.videoStream = camera
                        _this.audioStream = mp3Stream


                        _this.audioPlayer = await document.querySelector('audio')
                        _this.audioPlayer.volume = 1;
                        _this.audioPlayer.srcObject = mp3Stream;
                        _this.selectedAudio = mp3Stream

                        _this.showPopup = false
                        console.log('camera',camera)
                        console.log('mp3Stream',mp3Stream)
                    });
                });

            },

            async selectAudio(item) {
                var _this = this
                _this.isLoading = true;
                if(this.selectedAudio == null) {                    
                    navigator.mediaDevices.getUserMedia({
                        video: {
                            width: _this.width,
                            height: _this.height
                        }
                    }).then(async function(camera) {
                        _this.video.muted = true;
                        _this.video.volume = 0;                        
                        _this.video.srcObject = camera;
                        _this.videoStream = camera


                        _this.audioPlayer = await document.querySelector('audio')
                        _this.audioPlayer.volume = 1;
                        _this.audioPlayer.src = item.url;
                        // _this.audioPlayer = new Audio(item.url);
                        await _this.audioPlayer.load()
                        _this.audioPlayer.addEventListener('canplay', async () => {
                            console.log('canplay')
                            try {
                                let stream;
                                const fps = 0;
                                if (_this.audioPlayer.captureStream) {
                                    stream = await _this.audioPlayer.captureStream(fps);
                                } else if (_this.audioPlayer.mozCaptureStream) {
                                    stream = await _this.audioPlayer.mozCaptureStream(fps);
                                } else {
                                    console.error('Stream capture is not supported');
                                    stream = null;
                                }
                                _this.audioStream = stream;
                                _this.selectedAudio = _this.audioStream
                                _this.isLoading = false;
                            } catch (error) {
                                console.log('error',error)
                            }
                            
                        });

                        // _this.audioStream = await _this.audioPlayer.captureStream();
                        // _this.selectedAudio = _this.audioStream

                        // _this.audioPlayer = new Audio(item.url);
                        // _this.audioPlayer.play()
                        // _this.audioStream = await _this.audioPlayer.captureStream();   
                        // _this.audioPlayer.volume = 1;
                        // _this.selectedAudio = _this.audioStream

                        // console.log('_this.audioPlayerrrr',_this.audioPlayer)

                        // _this.audioStream.onactive = function(e){
                        //     console.log('The video is active',e);
                        // };

                        _this.showPopup = false
                        _this.isLoading = false;
                        
                        // console.log('camera',camera)
                        // console.log('33',_this.audioStream)
                    });
                }
            },

            showMusic() {
                this.showPopup = true
                this.audioStream = null
                console.log('showMusic')
            },

            backRecord() {
                this.$router.go('-1')
                // this.showBack = false
                // this.showRecord = true
                // this.showStop = false                
                // this.showPlay = false
                // this.showNext = false

                // this.dataAvailable = false
                // this.video.pause()
                // this.audioPlayer.pause()
                // this.selectedAudio = null

                // this.initiateCamera()

                
            },

            async startRecord() {
                console.log('this.audioPlayer',this.audioPlayer)

                if(this.audioPlayer == null) {
                    this.recorder = await RecordRTC(this.videoStream, {
                        type: 'video',
                        // video: {
                        //     width: this.width,
                        //     height: this.height
                        // }
                    });
                    console.log('this.videoStream',this.videoStream)

                } else {
                    var finalStream = new MediaStream();
                
                    if(this.audioStream!==null) {
                        await getTracks(this.audioStream, 'audio').forEach( async function(track) {
                            await finalStream.addTrack(track);
                        });
                    }

                    if(this.videoStream!==null) {
                        await getTracks(this.videoStream, 'video').forEach(async function(track) {
                            await finalStream.addTrack(track);
                        });
                    }                

                    this.recorder = await RecordRTC(finalStream, {
                        type: 'video',
                        // video: {
                        //     width: this.width,
                        //     height: this.height
                        // }
                    });
                }

                this.recorder.camera = this.videoStream;

                this.showStop = true
                this.showRecord = false
                this.showPlay = false

                if(this.audioPlayer!==null) {
                    this.audioPlayer.play()
                }
                
                this.recorder.startRecording();

                // var secondsLeft = 30;
                // (function looper() {
                //     secondsLeft--;

                //     if(secondsLeft < 0) {
                //         updateMediaHTML('Mixed Microphone+Mp3!');
                //         return;
                //     }
                //     updateMediaHTML('Seconds left: ' + secondsLeft);
                //     setTimeout(looper, 1000);
                // })();
                
                setTimeout(function(t) {
                    t.stopRecord()
                }, 10000,this)
            },
            
            stopRecordingCallback() {
                console.log('here',URL.createObjectURL(this.recorder.getBlob()))
                this.recordedData = this.recorder.getBlob()
                this.video.src = this.video.srcObject = null;
                this.video.muted = false;
                this.video.volume = 1;
                this.video.autoplay = false
                // this.video.width= this.width                
                // this.video.height= this.height
                this.video.src = URL.createObjectURL(this.recorder.getBlob());

                this.dataAvailable=true,
                this.recordedData = this.recorder.getBlob()
                if(this.audioPlayer!==null) {
                    this.audioPlayer.pause()
                    this.audioPlayer.src = this.audioPlayer.srcObject = null;
                }
                this.selectedAudio = null
                
                this.recorder.camera.stop();
                this.recorder.destroy();
                this.recorder = null;
            },

            stopRecord() {
                this.showStop = false
                this.showRecord = false
                this.showPlay = true
                if(this.recorder!==null) {
                    this.recorder.stopRecording(this.stopRecordingCallback);
                }
                
            },

            playRecord() {
                this.showStop = false
                this.showRecord = false
                this.showPlay = true
                this.video.play();
                if(this.dataAvailable) {
                    this.showNext = true
                    this.showBack = true
                }
            },

            async post() {
                this.isLoading = true

                var file = await this.axios.put('https://test4.services-money91.com/upload_file/binary?fileType=webm', this.recordedData, {
                    headers: {
                        "Content-Type": "application/octet-stream",
                        "X-Auth-Id": this.xauthid,
                        "JWT-TOKEN": this.jwtToken
                    }
                })

                if (file.data.Success) {
                    let res = await this.axios.post(
                    `${window.serverUrl}api/uploadvideo`,
                    {
                        userId: this.user.id,
                        xauthtoken:this.xauthtoken,
                        name:null,
                        poster: null,
                        url: file.data.Data.CdnUrl

                    });

                    if (res !== null && res.data.status) {
                        this.isLoading = false;
                        
                    } else {
                        this.$toast.bottom(res.data.message);
                    }

                    this.$router.go('-1')
                    
                } else {
                    this.isLoading = false
                    console.log("not uploaded!");
                }
            }
        },

        beforeDestroy() { 
            if (this.player) {
                // this.player.dispose();
            }
        }
    }
</script>

<style scoped> 
.popup {
    position: absolute;
    z-index: 9;
    top: 10%;
    background: #fff;
    width: 100%;
    padding: 15px;
}
.song{
    padding: 10px 0;
}
.close {
    text-align: center;
}
.record {
    position: absolute;
    z-index: 9;
    bottom: 10%;
    left: 40%;
    border-radius: 50%;
    height: 60px;
    width: 60px;
}
.back, .addmusic {
    position: absolute;
    z-index: 9;
    bottom: 10%;
    left: 10%;
    border-radius: 50%;
    height: 60px;
    width: 60px;
}
.next {
    position: absolute;
    z-index: 9;
    bottom: 10%;
    right: 10%;
    border-radius: 50%;
    height: 60px;
    width: 60px;
}
.video-js .vjs-tech {
    object-fit: cover;
}


</style>