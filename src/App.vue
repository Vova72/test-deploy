<template>

  <body>
    <div class="header">
      <div class="logo">
        <div class="header__back">
          <i class="fas fa-angle-left"></i>
        </div>
        <h2 style="color: white; font-size: 30px">Video meet</h2>
      </div>
    </div>  
    <div class="main">  
    <div class="main__left">
      <div class="videos__group">
        <div id="video-grid">
          <div id="app">
    
    

    <vue-webrtc id="call-canvas" width="100%" :roomId="roomId" ref="webrtc" v-on:share-started="shareStarted"  v-on:share-stopped="leftRoom" v-on:left-room="leftRoom" v-on:joined-room="joinedRoom"/>
    
   
  </div>
        </div>
      </div>
      <div class="options">
        <div class="options__left" style="margin-top: -40px">
          <div id="app" class="options__button" style="padding: 3px 5px; ">
            <button type="button"  style="background: none; border: none; color: white; font-family: 'Poppins', sans-serif;" id="join-btn" @click="toggleRoom">{{hasJoined ? 'Leave Room' : 'Join Room'}}</button>
          </div>
          <div id="app" class="options__button" style="padding: 3px 5px; ">
            <button type="button" id="screen-share-btn" @click="screenShare" v-if="hasJoined"  style="background: none; border: none; color: white; font-family: 'Poppins', sans-serif;">Screen Share</button>
          </div>
          <div id="app" class="options__button" style="padding: 3px 5px; ">
            <button @click="copyClipboard" style="background: none; border: none; color: white; font-family: 'Poppins', sans-serif;">Share Meeting</button>
          </div>
        </div>
        <div class="options__right"  style="margin-top: -40px">
          <div id="app" class="options__button" style="padding: 0px 75px; ">
            <input style="background: white; border: none; color: blakc; font-family: 'Poppins', sans-serif; padding-left: 3px; border-radius: 3px" placeholder="Enter room ID" id="room-input"/>
          </div>
        </div>
      </div>
    </div>
    <div class="main__right">
      <div class="main__in_window">
          <div class="insts">
            <h2 style="color: #2f80ec; font-size: 30px">Instruction</h2>
            <br>
            <Span>
            <p>1. Enter the id of the room you want to join</p>
            <br>
            <p>2. Next, press the button "join room" to connect</p>
            <br>
            <p>3. Then you can use all the functionality of the site you need</p>
            <br>
            </Span> 
            <br>
            <h2 style="color: #2f80ec; font-size: 30px">Functional</h2>
            <br>
            <Span>
              <p>1. Screen demonstration</p>
              <br>
            <p>2. Change the user volume</p>
              <br>
            <p>3. Stop video</p>
            </Span>
            <br>
            <h2 style="color: #2f80ec; font-size: 30px">Support</h2>
            <br>
            <Span>
              <br>
              <span style="display: flex; justify-content: center; flex-direction: row">
              <img height="56px" width="56px" src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/ec/Circle-icons-mail.svg/1200px-Circle-icons-mail.svg.png" alt="" style="margin: 5px 20px 0 0">
              <img height="54px" width="54px" src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/83/Telegram_2019_Logo.svg/1024px-Telegram_2019_Logo.svg.png" alt="" style="margin: 5px 20px 0 20px">
              <img height="70px" width="70px" src="https://freepngimg.com/save/62860-logo-twitter-computer-icons-free-photo-png/512x512" alt="" style="margin: 0px 20px 0 20px">
              </span>
              
            </Span>
          </div>
      </div>
      
    </div>
  </div>

  </body>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      roomId: 'roomId',
      hasJoined: false,
      mediaRecorder: null,
      chunks: [],
      userStream: null
    }
  },
  mounted () {
    const hash =  window.location.hash
    if(hash != '') {
      this.roomId = hash.substring(1)
      this.toggleRoom()
    }
  },
  methods: {
    onStop () {
      var blob = new Blob(this.chunks, { 'type' : 'video/webm' }); // other types are available such as 'video/webm' for instance, see the doc for more info
      this.chunks = [];
      const file = new File ([blob], 'meeting.webm', { 'type' : 'video/webm' })
      console.log(file)
      var a = document.createElement("a"),
                url = URL.createObjectURL(file);
        a.href = url;
        a.download = 'meeting.webm';
        document.body.appendChild(a);
        a.click()
    },
    pushData (e) {
      this.chunks.push(e.data);
      console.log(e.data)
    },
    record () {
      this.mediaRecorder.start()
    },
    async toggleRoom () {
      try {
        if(this.hasJoined) {
          this.$refs.webrtc.leave()
          this.hasJoined = false
          this.mediaRecorder.stop()
        } else {
          await this.$refs.webrtc.join()
          this.userStream = this.$refs.webrtc.videoList[0].stream
          this.mediaRecorder = new MediaRecorder(this.userStream)
          this.mediaRecorder.ondataavailable = e => this.pushData(e)
          this.mediaRecorder.onstop = () => this.onStop()
          this.hasJoined = true

        }
      } catch (e) {
        console.log(e)
      }

    },
    screenShare () {
      this.$refs.webrtc.shareScreen()
    },
    async addTrack(streamId) {
      try {
        console.log(streamId)
        const streams = this.$refs.webrtc.videoList
        console.log(streams)
        streams.forEach(stream => {
          console.log(stream.getTracks())
          this.userStream.addTrack(stream)
        })
      } catch (e) {
          console.log(e)
      }
    },
    joinedRoom (streamId) {
      // cт айди
      console.log(streamId)
    },
    shareStarted (streamId) {
      console.log(streamId)
      // + ст айди
    },
    
    async copyClipboard () {
      await navigator.clipboard.writeText(`https://localhost/${this.roomId}`)
      alert('Link copied to clipboard!')
    }
  }
}
</script>

<style>

.insts h2{
  display: flex;
  justify-content: center
}

.insts span{
  margin-left: 10px;
  
  flex-direction: column;
  opacity: .8;
  display: flex;
  justify-content: center;
  color: white;
  font-size: 20px;
  font-family: "Poppins", sans-serif;
}

@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600&display=swap");

:root {
  --main-darklg: #1d2635;
  --main-dark: #161d29;
  --primary-color: #2f80ec;
  --main-light: #eeeeee;
  font-family: "Poppins", sans-serif;
}

* {
  font-family: "Poppins", sans-serif;
  margin: 0;
  padding: 0;
}

.header {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 8vh;
  position: relative;
  width: 100%;
  background-color: var(--main-darklg);
}

.logo > h3 {
  color: var(--main-light);
}

.main {
  overflow: hidden;
  height: 92vh;
  display: flex;
}

.main__left {
  flex: 0.7;
  display: flex;
  flex-direction: column;
}

.videos__group {
  flex-grow: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
  background-color: var(--main-dark);
}

video {
  height: 300px;
  border-radius: 1rem;
  margin: 0.5rem;
  width: 400px;
  object-fit: cover;
  transform:rotateX(-180deg);
}

.options {
  padding: 1rem;
  display: flex;
  justify-items: center;
  background-color: var(--main-darklg);
}

.options__left {
  display: flex;
}

.options__right {
  margin-left: auto;
}

.options__button {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: var(--primary-color);
  height: 50px;
  border-radius: 5px;
  color: var(--main-light);
  font-size: 1.2rem;
  width: 50px;
  margin: 0 0.5rem;
}

.background__red {
  background-color: #f6484a;
}

.main__right {
  display: flex;
  flex-direction: column;
  flex: 0.3;
  background-color: #242f41;
}

.main__in_window {
  flex-grow: 1;
  overflow-y: scroll;
}

.main__in_window::-webkit-scrollbar {
  display: none;
}

.main__inst_container {
  padding: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.main__inst_container > input {
  height: 50px;
  flex: 1;
  font-size: 1rem;
  border-radius: 5px;
  padding-left: 20px;
  border: none;
}

.insts {
  display: flex;
  flex-direction: column;
  margin: 1.5rem;
}

.inst {
  display: flex;
  flex-direction: column;
}

.inst > b {
  color: #eeeeee;
  display: flex;
  align-items: center;
  text-transform: capitalize;
}

.inst > b > i {
  margin-right: 0.7rem;
  font-size: 1.5rem;
}
#room-input {
  margin-bottom: 3px;
}
#call-canvas {
  background-color: transparent;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
#join-btn {
  margin: 4px 2px;
}
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600&display=swap");

:root {
  --main-darklg: #1d2635;
  --main-dark: #161d29;
  --primary-color: #2f80ec;
  --main-light: #eeeeee;
  font-family: "Poppins", sans-serif;
}

* {
  margin: 0;
  padding: 0;
}

.header {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 8vh;
  position: relative;
  width: 100%;
  background-color: var(--main-darklg);
}

.logo > h3 {
  color: var(--main-light);
}

.main {
  overflow: hidden;
  height: 92vh;
  display: flex;
}

.main__left {
  transform: rotateY(180deg);
  -webkit-transform: rotateY(180deg);
  -moz-transform: rotateY(180deg);
  flex: 0.7;
  display: flex;
  flex-direction: column;
}

.videos__group {
  flex-grow: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
  background-color: var(--main-dark);
}

video {
  height: 300px;
  border-radius: 1rem;
  margin: 0.5rem;
  width: 400px;
  object-fit: cover;
  transform: rotateY(180deg);
  -webkit-transform: rotateY(180deg);
  -moz-transform: rotateY(180deg);
  
}

.options {
  transform: rotateY(180deg);
  -webkit-transform: rotateY(180deg);
  -moz-transform: rotateY(180deg);
  padding: 1rem;
  display: flex;
  background-color: var(--main-darklg);
}

.options__left {
  display: flex;
}

.options__right {
  margin-left: auto;
}

.options__button {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: var(--primary-color);
  height: 50px;
  border-radius: 5px;
  color: var(--main-light);
  font-size: 1.2rem;
  width: 50px;
  margin: 0 0.5rem;
}

.background__red {
  background-color: #f6484a;
}

.main__right {
  display: flex;
  flex-direction: column;
  flex: 0.3;
  background-color: #242f41;
}

.main__in_window {
  flex-grow: 1;
  overflow-y: scroll;
}

.main__in_window::-webkit-scrollbar {
  display: none;
}

.main__inst_container {
  padding: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.main__inst_container > input {
  height: 50px;
  flex: 1;
  font-size: 1rem;
  border-radius: 5px;
  padding-left: 20px;
  border: none;
}

.insts {
  display: flex;
  flex-direction: column;
  margin: 1.5rem;
}

.inst {
  display: flex;
  flex-direction: column;
}

.inst > b {
  color: #eeeeee;
  display: flex;
  align-items: center;
  text-transform: capitalize;
}

.inst > b > i {
  margin-right: 0.7rem;
  font-size: 1.5rem;
}

.inst > span {
  background-color: #eeeeee;
  margin: 1rem 0;
  padding: 1rem;
  border-radius: 5px;
}

#video-grid {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
}

#showin {
  display: none;
}

.header__back {
  display: none;
  position: absolute;
  font-size: 1.3rem;
  top: 17px;
  left: 28px;
  color: #fff;
}

@media (max-width: 700px) {
  .main__right {
    display: none;
  }
  .main__left {
    width: 100%;
    flex: 1;
  }

  video {
    height: auto;
    width: 100%;
  }

  #showin {
    display: flex;
  }
}


</style>
