<template>
  <EpUploader type="button"
              :options="options"
              :file-status-text="statusText"
              class="uploader-example"
              ref="uploader"
              @file-complete="fileComplete"
              @file-success="fileSuccess"
              @complete="complete"
              :attrs="attrs"
              :msg="msg"
              ></EpUploader>
</template>

<script>
  const baseUrl = 'http://localhost:8030'
  export default {
    data () {
      return {
        options: {
          target: '//localhost:8089/boot/uploader/chunk', // '//jsonplaceholder.typicode.com/posts/',
          // target: baseUrl + '/core-api/eemp/v1/uploader',
          testChunks: false
        },
        attrs: {
          accept: '*'
        },
        statusText: {
          success: '成功了',
          error: '出错了',
          uploading: '上传中',
          paused: '暂停中',
          waiting: '等待中'
        },
        msg: {
          code: 100,
          msg: '1'
        }
      }
    },
    methods: {
      complete () {
        console.log('complete', arguments)
      },
      fileComplete () {
        console.log('file complete', arguments)
      },
      fileSuccess (rootFile, file, message, chunk) {
        let mes = JSON.parse(message)
        if (mes.code === 500) {
          this.msg.code = mes.code
          this.msg.msg = mes.message
        }
      }
    },
    mounted () {
      this.$nextTick(() => {
        window.uploader = this.$refs.uploader.uploader
      })
    }
  }
</script>

<style>
  .uploader-example {
    width: 880px;
    padding: 15px;
    margin: 40px auto 0;
    font-size: 12px;
    box-shadow: 0 0 10px rgba(0, 0, 0, .4);
  }
  .uploader-example .uploader-btn {
    margin-right: 4px;
  }
  .uploader-example .uploader-list {
    max-height: 440px;
    overflow: auto;
    overflow-x: hidden;
    overflow-y: auto;
  }
</style>
