
<template lang="pug">
  div
    input#fileUploader(ref="fileUploader" type="file", accept="image/*", @change="detectFiles($event.target.files)")
    q-input.full-width(
      readonly
      label="Upload picture"
      @click="trigger"
      :value="fileName"
      :after="[{icon, handler: trigger}]"
      stack-label="Select file to upload"
      :error="error"
      :loading="loading"
    )
</template>

<script>
export default {
  name: 'u-file-uploader',
  props: ['error'],
  data () {
    return {
      progressUpload: 0,
      file: File,
      uploadTask: '',
      downloadURL: '',
      fileName: '',
      loading: false,
      icon: 'mdi-library-plus'
    }
  },
  methods: {
    detectFiles (fileList) {
      this.fileName = this.$refs.fileUploader.files.item(0).name
      Array.from(Array(fileList.length).keys()).map(x => {
        this.upload(fileList[x])
      })
    },
    trigger () {
      this.$refs.fileUploader.click()
    },
    upload () {
      const vm = this
      vm.uploadTask = vm.$firebaseStorage.child(`images/projects/${Date.now()}`).put(vm.$refs.fileUploader.files.item(0))
      vm.loading = true
      vm.uploadTask.then(function (snapshot) {
        vm.uploadTask.snapshot.ref.getDownloadURL().then(function (downloadURL) {
          vm.downloadURL = downloadURL
          vm.loading = false
          vm.$emit('upload', downloadURL)
        }).catch((err) => {
          vm.loading = false
          vm.$q.notify(err.message)
        })
      }).catch((err) => {
        vm.loading = false
        vm.$q.notify(err.message)
      })
    }
  },
  mounted () {
  },
  computed: {
  },
  watch: {
    uploadTask () {
      this.uploadTask.on('state_changed', function (sp) {
        this.progressUpload = Math.floor(sp.bytesTransferred / sp.totalBytes * 100)
      })
    },
    loading () {
      this.loading ? this.icon = '' : this.icon = 'mdi-library-plus'
    }
  }
}
</script>

<style lang="stylus">
.progress-bar {
  margin 10px 0
}
input {
  display none
}
</style>
