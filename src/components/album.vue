<template>
  <div class="playlist">
    <div class="left">
      <img :src="images[0].url" alt="">
      // change it to the component later
      <h2>{{playListName}}</h2>
      <div class="owner">
        <span>By</span>
        <a href="">{{owner[0].name}}</a>
      </div>
      <p class="song-count">{{tracks.length}} songs</p>
      <button class="btn btn-green">play</button>
    </div>
    <div right="right" >
      <track-display v-for="(track,index) in tracks" :track-title="track.name" :artist="track.artists" :trackNum="index" v-on:togglePlay="togglePlayState"></track-display>
    </div>
  </div>
</template>

<script>
export default {
  name: 'playlist',
  data () {
    return {
      playListName: '',
      owner: '',
      tracks: [],
      images: '',
      isPlaying: false
    }
  },
  methods: {
    getPlayList() {
      var albumId = location.href.split('/').slice(-1)[0];
      console.log('albumId', albumId);
      this.$http.get('https://api.spotify.com/v1/albums/' + albumId, {headers: {
        Authorization: 'Bearer ' + this.getLocalStorage('spotify-token')
      }})
      .then(function(res) {
        this.playListName = res.body.name;
        this.tracks = res.body.tracks.items;
        this.owner = res.body.artists;
        this.images = res.body.images;
        // console.log(this.images[0].url);
        console.log('this.tracks',this.tracks);
        console.log('res', res.body);


      }, function(error) {
        // console.log(error.bodyText);
        var errorContent = JSON.parse(error.bodyText);
        if (errorContent.error.message === 'The access token expired') {
          var authUrl = 'https://accounts.spotify.com/authorize?client_id=fb0e57d017fe4c1b817fd5f624ebdf4d&redirect_uri=http:%2F%2Flocalhost:8080%2Fcallback&scope=user-read-private%20user-read-email&response_type=token&state=123';
          this.setLocalStorage('previous-url', location.href);
          window.location.href = authUrl;
        }
      })
    },
    togglePlayState(trackNum) {
      console.log('state toggled!', trackNum);
      this.$emit('togglePlay', this.tracks, trackNum);
      console.log(this.tracks);
    }
  },
  created: function() {
    this.getPlayList();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.song-count {
  text-transform: uppercase;
}
</style>
