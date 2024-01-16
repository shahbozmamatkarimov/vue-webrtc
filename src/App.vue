<template>
  <div id="app">
    <div class="flex backdrop-blur-md bg-[#2E2C54] mb-4 pb-4">
        <div class="w-full content-center">
          <form class="rounded pt-6 pb-8 mb-4">
            <div class="mb-4">
              <label class="block text-white text-sm font-bold mb-2" for="room-input">
                Chat ID
              </label>
              <input v-model="roomId" placeholder="Enter room ID" id="room-input" class="shadow appearance-none border rounded w-full py-2 px-3 bg-[#8871E6] text-white font-bold text-lg leading-tight focus:outline-none focus:shadow-outline"/>
            </div>
          <div class="flex items-center justify-between"> 
            <button type="button" @click="copyClipboard" class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded">Chat linki</button>
            <button type="button" id="join-btn" @click="toggleRoom" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">{{hasJoined ? 'Tark etish' : 'Chatga kirish'}}</button>
            <button type="button" id="screen-share-btn" @click="screenShare" v-if="hasJoined" class="bg-purple-500 hover:bg-purple-700 text-white font-bold py-2 px-4 rounded">Screen Share</button>

            <button type="button" @click="record" v-if="hasJoined" class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded">Record</button>
          </div>
          <vue-webrtc id="call-canvas" :roomId="roomId" ref="webrtc" v-on:share-started="shareStarted"  class="w-full" v-on:share-stopped="leftRoom" v-on:left-room="leftRoom" v-on:joined-room="joinedRoom" width="100%"/>
        </form>
      </div>
    </div>


  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      roomId: 'roomId',
      hasJoined: false,
      mediaRecorder: {},
      chunks: [],
      userStream: {}
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
      var a = document.createElement("a"),
                url = URL.createObjectURL(file);
        a.href = url;
        a.download = 'meeting.webm';
        document.body.appendChild(a);
        a.click()
    },
    pushData (e) {
      this.chunks.push(e.data);
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
        alert(e)
      }

    },
    screenShare () {
      try {
        this.$refs.webrtc.shareScreen()
      } catch (e) {
        alert('Screen share not available')
      }
    },
    async addTrack() {
      try {
        const streams = this.$refs.webrtc.videoList
        console.log(streams)
        streams.forEach(stream => {
          this.userStream.addTrack(stream)
        })
      } catch (e) {
          alert(e)
      }
    },
    joinedRoom (streamId) {
      // this.addTrack(streamId)
      console.log(streamId)
    },
    shareStarted (streamId) {
      console.log(streamId)
      // this.addTrack(streamId)
    },
    leftRoom (streamId) {
      // this.mediaRecorder.removeTrack(streamId)
      console.log(streamId)
    },
    async copyClipboard () {
      await navigator.clipboard.writeText(`https://vue-webrtc-two.vercel.app/#${this.roomId}`)
      alert('Link copied to clipboard!')
    },
    async share () {
    const shareData = {
        title: 'J Comp Meet',
        text: 'Join my meeting!',
        url: `https://meet.jcompsolu.com/#${this.roomId}`
      }
      try {
      await navigator.share(shareData)
      } catch(err) {
      this.copyClipboard()
      }
    }
  }
}
</script>

<style>
#room-input {
  margin-bottom: 3px;
}
#call-canvas {
  background-color: transparent;
}

#app {
  background: #242247;
  height: 100vh;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#join-btn {
  margin: 4px 2px;
}

img {
  height: 80px;
  width: 100%;
}
</style>
