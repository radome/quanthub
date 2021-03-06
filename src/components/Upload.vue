<template>
  <div class="upload container-fluid">
    <alert ref='alert'></alert>
    <form enctype="multipart/form-data" method="post" action="#" v-on:submit.prevent="upload">
      <div class="form-group">
        <div class="form-group">
          <label for="quantType">Select a Quant Type:</label>
          <select id="quant-type" class="form-control" name="quantType" v-model="quantType">
              <option :value="null" selected disabled>Please select a quant type ...</option>
              <option v-for="(option, key) in quantTypes" v-bind:key="key" v-bind:value="key">{{option.name}}</option>
          </select>
        </div>
        <input type="file" name="file-input" id="file-input" ref="fileInput" class="file" v-on:change.prevent="addFilenames">
        <div class="input-group">
          <input class="form-control" type="text" disabled placeholder="Upload File..." ref="browseFiles">
          <span class="input-group-btn">
            <button class="btn btn-success spacer" v-on:click.prevent="browseFiles" type="button">Browse</button>
            <button name="submit" class="btn btn-success" type="submit">Upload</button>
          </span>
        </div>
      </div>
    </form>
  </div>
</template>

<script>

// Uploads a file. Parse options dependent on quantType
import Vue from 'vue'
import QuantFile from '@/components/QuantFile'
import quantTypes from '@/config/quantTypes'
import Alert from '@/components/Alert'

export default {
  name: 'Upload',
  props: {
  },
  data () {
    return {
      msg: 'Upload',
      Cmp: Vue.extend(QuantFile),
      quantType: null,
      quantTypes: quantTypes,
      filename: null,
      validFileExtensions: ['.csv', '.txt']
    }
  },
  computed: {
    filename_filtered () {
      return this.filename ? this.filename.replace(/^.*[\\]/, '') : null
    }
  },
  components: {
    QuantFile,
    Alert
  },
  methods: {
    valid_filetype () {
      var typeValid = false
      var sFilename = this.filename_filtered
      if (sFilename && sFilename.length > 0) {
          for (var j = 0; j < this.validFileExtensions.length; j++) {
              var sCurExtension = this.validFileExtensions[j]
              if (sFilename.substr(sFilename.length - sCurExtension.length, sCurExtension.length).toLowerCase() == sCurExtension.toLowerCase()) {
                  typeValid = true
                  break
              }
          }
      }
      return typeValid
    },
    // create a quantFile based on the quantType
    // if the upload is successful it is saved to local storage
    // The user is then redirected to the plate page
    // where the file is retrieved from local storage.
    async upload () {
      if (!this.quantType) {
        this.$refs.alert.show(`Please select a quant type and file before uploading!`, 'warning')
        return
      }
      if(!this.valid_filetype()) {
        this.$refs.alert.show(`Please select a csv file before uploading!`, 'warning')
        return
      }
      const file = document.getElementById('file-input').files[0]
      let quantFile = new this.Cmp({propsData: {quant: this.quantType, filename: this.filename_filtered}})
      quantFile.upload(file)
        .then(() => {
          localStorage.setItem(quantFile.id, JSON.stringify(quantFile.json))
          this.$router.push({ path: `/plate/${quantFile.id}` })
        })
        .catch((error) => {
          var msg = 'File rejected, please check the file and then retry, reason: '
          switch (typeof error) {
            case 'object':
              switch (error.name) {
                case 'QuotaExceededError':
                  msg = 'Local storage is full up, please clear and then retry'
                  break
                case 'TypeError':
                  msg += 'Formatting of file incorrect'
                  break
                default:
                  msg += `Exception message: ${error.message}`
              }
              break
            case 'string':
              msg += `Error message: ${error}`
              break
            default:
              msg += `Unrecognised error type: ${error}`;
          }
          this.$refs.alert.show(msg, 'danger')
          /*eslint no-console: ["error", { allow: ["error"] }] */
          console.error('rejected:', error)
        })
    },
    browseFiles () {
      this.$refs.fileInput.click()
    },
    addFilenames () {
      /*eslint no-console: */
      this.filename = this.$refs.fileInput.value
      this.$refs.browseFiles.value = this.filename_filtered
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
