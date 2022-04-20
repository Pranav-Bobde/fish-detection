<template>
  <q-page class="constrain-more q-pa-md">
    <div class="text-center">
      <h1>
        FISH DETECTION
      </h1> 
    </div>
    <div style="width: 50%; margin: auto;">
      <div class="camera-frame q-pa-md">
        <video 
          v-show="!imageCaptured"
          ref="video"
          class="full-width"
          autoplay
        />
        <canvas
          v-show="imageCaptured"
          ref="canvas"
          class="full-width"
          height="240"
        />
      </div>
    </div>
    <div style="width: 50%; margin: auto;">
      <div class="text-center q-pa-md">
        <q-btn
          v-if="hasCameraSupport"
          @click="captureImage"
          :disable="imageCaptured"
          color="grey-10"
          icon="eva-camera"
          size="lg"
          round
        />
        <q-file
          v-else
          v-model="imageUpload"
          @input="captureImageFallback"
          label="CHOOSE AN IMAGE"
          label-color="black"
          accept="image/*"
          outlined
        >
          <template v-slot:prepend>
            <q-icon name="attach_file" />
          </template>
        </q-file>
      </div>
    </div>
      <div class="row justify-center q-mt-lg">
        <q-btn
          class="text-h6"
          @click="addPost()"
          :disable="!post.photo"
          color="primary"
          label="Upload Image"
          rounded
        />
      </div>
  </q-page>
</template>

<script>
import { uid } from 'quasar'
require('md-gum-polyfill')

export default {
  name: 'PageCamera',
  data() {
    return {
      post: {
        photo: null
      },
      imageCaptured: false,
      imageUpload: [],
      hasCameraSupport: true,
    }
  },
  computed: {
    backgroundSyncSupported() {
      if ('serviceWorker' in navigator && 'SyncManager' in window) return true
      return false
    }
  },
  methods: {
    displayRes(data){
      console.log('here')
      this.$q.dialog({
        title: 'PREDICTION',
        message: `${data.fish_species} -> ${data.prediction}`,
        ok: {
          push: true
        },
        persistent: true
      }).onOk(() => {
        // console.log('>>>> OK')
      })
    },
    initCamera() {
      navigator.mediaDevices.getUserMedia({
        video: true
      }).then(stream => {
        this.$refs.video.srcObject = stream
      }).catch(error => {
        this.hasCameraSupport = false
      })
    },
    captureImage() {
      let video = this.$refs.video
      let canvas = this.$refs.canvas
      canvas.width = video.getBoundingClientRect().width
      canvas.height = video.getBoundingClientRect().height
      let context = canvas.getContext('2d')
      context.drawImage(video, 0, 0, canvas.width, canvas.height)
      this.imageCaptured = true
      this.post.photo = this.dataURItoBlob(canvas.toDataURL())
      this.disableCamera()
    },
    captureImageFallback(file) {
      this.post.photo = file

      let canvas = this.$refs.canvas
      let context = canvas.getContext('2d')

      var reader = new FileReader()
      reader.onload = event => {
        var img = new Image()
        img.onload = () => {
          canvas.width = img.width
          canvas.height = img.height
          context.drawImage(img,0,0)
          this.imageCaptured = true
        }
        img.src = event.target.result
      }
      reader.readAsDataURL(file)

    },
    disableCamera() {
      this.$refs.video.srcObject.getVideoTracks().forEach(track => {
        track.stop()
      })
    },
    dataURItoBlob(dataURI) {
      // convert base64 to raw binary data held in a string
      // doesn't handle URLEncoded DataURIs - see SO answer #6850276 for code that does this
      var byteString = atob(dataURI.split(',')[1]);

      // separate out the mime component
      var mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0]

      // write the bytes of the string to an ArrayBuffer
      var ab = new ArrayBuffer(byteString.length);

      // create a view into the buffer
      var ia = new Uint8Array(ab);

      // set the bytes of the buffer to the correct values
      for (var i = 0; i < byteString.length; i++) {
          ia[i] = byteString.charCodeAt(i);
      }

      // write the ArrayBuffer to a blob, and you're done
      var blob = new Blob([ab], {type: mimeString});
      return blob;

    },
    addPost() {
      let formData = new FormData();
      formData.append("file", this.post.photo, "pic.jpeg");
      this.$axios
        .post("http://127.0.0.1:5000/predict", formData)
        .then(response => {
          console.log("response: ", response);
          let data = response.data
          
          this.displayRes(data)
          
        })
        .catch(err => {
          console.log(err);
        });
    }
  },
  mounted() {
    this.initCamera()
  },
  beforeDestroy() {
    if (this.hasCameraSupport) {
      this.disableCamera()
    }
  }
}
</script>

<style lang="sass">
  .camera-frame
    border: 2px solid $grey-10
    border-radius: 10px
</style>