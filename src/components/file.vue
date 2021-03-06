<template>
  <div class="uploader-file" :status="status" :msg="msg">
    <slot
      :file="file"
      :list="list"
      :status="status"
      :paused="paused"
      :error="error"
      :response="response"
      :average-speed="averageSpeed"
      :formated-average-speed="formatedAverageSpeed"
      :current-speed="currentSpeed"
      :is-complete="isComplete"
      :is-uploading="isUploading"
      :size="size"
      :formated-size="formatedSize"
      :uploaded-size="uploadedSize"
      :progress="progress"
      :progress-style="progressStyle"
      :progressing-class="progressingClass"
      :time-remaining="timeRemaining"
      :formated-time-remaining="formatedTimeRemaining"
      :type="type"
      :extension="extension"
      :file-category="fileCategory"
      >
      <!---自定义区域-->
      <div class="ep-uploader-file-list" :class="progressingClass">
        <!--文件展示区域-->
        <span >
          <i  class="ivu-icon" :class="format(file)"></i>
          <span class="ep-uploader-file-name" :attr="file.name">{{file.name}}</span>
          <span v-show="status === 'uploading'" style="margin-left: 5px;">
            <span class="">{{formatedAverageSpeed}}</span>
            <span class="">{{formatedTimeRemaining}}</span>
          </span>
          <span v-show="status === 'error' || status === 'paused'" :style="epStatusStyle">{{statusText}}</span>
        </span>
        <!--删除处理-->
        <i class="ivu-icon ivu-icon-ios-close ivu-upload-list-remove" @click="remove"></i>
        <!--进度条-->
        <div class="ep-uploader-progress ep-uploader-progress-normal ep-uploader-progress-show-info">
          <div class="ep-uploader-progress-outer">
            <div  class="ep-uploader-progress-inner">
              <div class="ep-uploader-progress-bg" :class="epUploaderProgressStatus" :style="epProgressStyle"></div>
            </div>
          </div>
          <span class="ep-uploader-progress-text">
            <span  class="ep-uploader-progress-text-inner">{{progressStyle.progress}}</span>
          </span>
          <span class="ep-uploader-progress-text ep-uploader-progress-action">
            <span class="uploader-file-pause" @click="pause"></span>
            <span class="uploader-file-resume" @click="resume">️</span>
            <span class="uploader-file-retry" @click="retry"></span>
            <span class="uploader-file-remove" @click="remove"></span>
          </span>
        </div>
      </div>
      <!---自定义区域结束-->
    </slot>
  </div>
</template>

<script>
  import Uploader from 'simple-uploader.js'
  import events from '../common/file-events'
  import { secondsToStr } from '../common/utils'

  const COMPONENT_NAME = 'uploader-file'
  // 文件小图标前缀
  const prefixCls = 'ivu-icon'

  export default {
    name: COMPONENT_NAME,
    props: {
      file: {
        type: Object,
        default () {
          return {}
        }
      },
      list: {
        type: Boolean,
        default: false
      },
      msg: {
        type: Object
      }
    },
    data () {
      return {
        response: null,
        paused: false,
        error: false,
        averageSpeed: 0,
        currentSpeed: 0,
        isComplete: false,
        isUploading: false,
        size: 0,
        formatedSize: '',
        uploadedSize: 0,
        progress: 0,
        timeRemaining: 0,
        type: '',
        extension: '',
        progressingClass: '',
        epUploaderProgressStatus: '', // 进度条样式
        iconType: '' // 文件小图标样式，默认演示文件
      }
    },
    computed: {
      fileCategory () {
        const extension = this.extension
        const isFolder = this.file.isFolder
        let type = isFolder ? 'folder' : 'unknown'
        const categoryMap = this.file.uploader.opts.categoryMap
        const typeMap = categoryMap || {
          image: ['gif', 'jpg', 'jpeg', 'png', 'bmp', 'webp'],
          video: ['mp4', 'm3u8', 'rmvb', 'avi', 'swf', '3gp', 'mkv', 'flv'],
          audio: ['mp3', 'wav', 'wma', 'ogg', 'aac', 'flac'],
          document: ['doc', 'txt', 'docx', 'pages', 'epub', 'pdf', 'numbers', 'csv', 'xls', 'xlsx', 'keynote', 'ppt', 'pptx']
        }
        Object.keys(typeMap).forEach((_type) => {
          const extensions = typeMap[_type]
          if (extensions.indexOf(extension) > -1) {
            type = _type
          }
        })
        return type
      },
      progressStyle () {
        const progress = Math.floor(this.progress * 100)
        const style = `translateX(${Math.floor(progress - 100)}%)`
        return {
          progress: `${progress}%`,
          webkitTransform: style,
          mozTransform: style,
          msTransform: style,
          transform: style
        }
      },
      epProgressStyle () {
        const progress = Math.floor(this.progress * 100)
        return {
          width: `${progress}%`
        }
      },
      epStatusStyle () {
        return {
          'margin-left': '5px',
          'color': this.status === 'error' ? 'rgb(230, 93, 12)' : (this.status === 'success' ? '#19be6b' : '#2d8cf0')
        }
      },
      formatedAverageSpeed () {
        return `${Uploader.utils.formatSize(this.averageSpeed)} / s`
      },
      status () {
        const isUploading = this.isUploading
        const isComplete = this.isComplete
        const isError = this.error
        const paused = this.paused
        if (isComplete) {
          return 'success'
        } else if (isError) {
          return 'error'
        } else if (isUploading) {
          return 'uploading'
        } else if (paused) {
          return 'paused'
        } else {
          return 'waiting'
        }
      },
      statusText () {
        const status = this.status
        const fileStatusText = this.file.uploader.fileStatusText
        let txt = status
        if (typeof fileStatusText === 'function') {
          txt = fileStatusText(status, this.response)
        } else {
          txt = fileStatusText[status]
        }
        return txt || status
      },
      formatedTimeRemaining () {
        const timeRemaining = this.timeRemaining
        const file = this.file
        if (timeRemaining === Number.POSITIVE_INFINITY || timeRemaining === 0) {
          return ''
        }
        let parsedTimeRemaining = secondsToStr(timeRemaining)
        const parseTimeRemaining = file.uploader.opts.parseTimeRemaining
        if (parseTimeRemaining) {
          parsedTimeRemaining = parseTimeRemaining(timeRemaining, parsedTimeRemaining)
        }
        return parsedTimeRemaining
      }
    },
    watch: {
      status (newStatus, oldStatus) {
        if (oldStatus && newStatus === 'uploading' && oldStatus !== 'uploading') {
          this.tid = setTimeout(() => {
            this.progressingClass = 'uploader-file-progressing'
          }, 200)
        } else {
          clearTimeout(this.tid)
          this.progressingClass = ''
        }
        if (newStatus === 'success') {
          this.epUploaderProgressStatus = 'ep-uploader-progress-success-bg'
        } else if (newStatus === 'error') {
          this.epUploaderProgressStatus = 'ep-uploader-progress-error-bg'
        } else {
          this.epUploaderProgressStatus = ''
        }
      }
    },
    methods: {
      _actionCheck () {
        this.paused = this.file.paused
        this.error = this.file.error
        this.isUploading = this.file.isUploading()
      },
      pause () {
        this.file.pause()
        this._actionCheck()
        this._fileProgress()
      },
      resume () {
        this.file.resume()
        this._actionCheck()
      },
      remove () {
        this.file.cancel()
      },
      retry () {
        this.file.retry()
        this._actionCheck()
      },
      processResponse (message) {
        let res = message
        try {
          res = JSON.parse(message)
        } catch (e) {}
        this.response = res
      },
      fileEventsHandler (event, args) {
        const rootFile = args[0]
        const file = args[1]
        const target = this.list ? rootFile : file
        if (this.file === target) {
          if (this.list && event === 'fileSuccess') {
            this.processResponse(args[2])
            return
          }
          this[`_${event}`].apply(this, args)
        }
      },
      _fileProgress () {
        this.progress = this.file.progress()
        this.averageSpeed = this.file.averageSpeed
        this.currentSpeed = this.file.currentSpeed
        this.timeRemaining = this.file.timeRemaining()
        this.uploadedSize = this.file.sizeUploaded()
        this._actionCheck()
      },
      _fileSuccess (rootFile, file, message) {
        if (this.msg.code === 500) {
          this._fileError(rootFile, file, message)
          console.error(this.msg.msg)
          return
        }
        if (rootFile) {
          this.processResponse(message)
        }
        this._fileProgress()
        this.error = false
        this.isComplete = true
        this.isUploading = false
      },
      _fileComplete (rootFile) {
        this._fileSuccess()
      },
      _fileError (rootFile, file, message) {
        this._fileProgress()
        this.processResponse(message)
        this.error = true
        this.isComplete = false
        this.isUploading = false
      },
      // 通过文件后缀名判断小图标样式，此项依赖于iview
      format (file) {
        const format = file.name.split('.').pop().toLocaleLowerCase() || ''
        let type = 'ios-document-outline'
        if (['gif', 'jpg', 'jpeg', 'png', 'bmp', 'webp'].indexOf(format) > -1) {
          type = 'ios-image'
        }
        if (['mp4', 'm3u8', 'rmvb', 'avi', 'swf', '3gp', 'mkv', 'flv'].indexOf(format) > -1) {
          type = 'ios-film'
        }
        if (['mp3', 'wav', 'wma', 'ogg', 'aac', 'flac'].indexOf(format) > -1) {
          type = 'ios-musical-notes'
        }
        if (['doc', 'txt', 'docx', 'pages', 'epub', 'pdf'].indexOf(format) > -1) {
          type = 'md-document'
        }
        if (['numbers', 'csv', 'xls', 'xlsx'].indexOf(format) > -1) {
          type = 'ios-stats'
        }
        if (['keynote', 'ppt', 'pptx'].indexOf(format) > -1) {
          type = 'ios-videocam'
        }
        return `${prefixCls}` + type
      },
      _isEmpty (obj) {
        return obj === null || obj === undefined || obj === ''
      }
    },
    mounted () {
      const staticProps = ['paused', 'error', 'averageSpeed', 'currentSpeed']
      const fnProps = [
        'isComplete',
        'isUploading',
        {
          key: 'size',
          fn: 'getSize'
        },
        {
          key: 'formatedSize',
          fn: 'getFormatSize'
        },
        {
          key: 'uploadedSize',
          fn: 'sizeUploaded'
        },
        'progress',
        'timeRemaining',
        {
          key: 'type',
          fn: 'getType'
        },
        {
          key: 'extension',
          fn: 'getExtension'
        }
      ]
      staticProps.forEach(prop => {
        this[prop] = this.file[prop]
      })
      fnProps.forEach((fnProp) => {
        if (typeof fnProp === 'string') {
          this[fnProp] = this.file[fnProp]()
        } else {
          this[fnProp.key] = this.file[fnProp.fn]()
        }
      })

      const handlers = this._handlers = {}
      const eventHandler = (event) => {
        handlers[event] = (...args) => {
          this.fileEventsHandler(event, args)
        }
        return handlers[event]
      }
      events.forEach((event) => {
        this.file.uploader.on(event, eventHandler(event))
      })
    },
    destroyed () {
      events.forEach((event) => {
        this.file.uploader.off(event, this._handlers[event])
      })
      this._handlers = null
    }
  }
</script>

<style>
  .uploader-file {
    position: relative;
    height: auto;
    /*line-height: 200px;*/
    overflow: hidden;
    border-bottom: 1px solid #cdcdcd;
  }
  .uploader-file[status="waiting"] .uploader-file-pause,
  .uploader-file[status="uploading"] .uploader-file-pause {
    display: block;
  }
  .uploader-file[status="paused"] .uploader-file-resume {
    display: block;
  }
  .uploader-file[status="error"] .uploader-file-retry {
    display: block;
  }
  .uploader-file[status="success"] .uploader-file-remove {
    display: none;
  }
  .uploader-file[status="error"] .uploader-file-progress {
    background: #ffe0e0;
  }
  .uploader-file[status="error"] .ep-uploader-progress-inner .ep-uploader-progress-bg {
    color: red !important;
  }
  .uploader-file-progress {
    position: absolute;
    width: 100%;
    height: 100%;
    background: #e2eeff;
    transform: translateX(-100%);
  }
  .uploader-file-progressing {
    transition: all .4s linear;
  }
  .uploader-file-info {
    position: relative;
    z-index: 1;
    height: 100%;
    overflow: hidden;
  }
  .uploader-file-info:hover {
    background-color: rgba(240, 240, 240, 0.2);
  }
  .uploader-file-info i,
  .uploader-file-info em {
    font-style: normal;
  }
  .uploader-file-name,
  .uploader-file-size,
  .uploader-file-meta,
  .uploader-file-status,
  .uploader-file-actions {
    float: left;
    position: relative;
    height: 100%;
  }
  .uploader-file-name {
    width: 45%;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    text-indent: 14px;
  }
  .uploader-file-icon {
    width: 24px;
    height: 24px;
    display: inline-block;
    vertical-align: top;
    margin-top: 13px;
    margin-right: 8px;
  }
  .uploader-file-icon::before {
    content: "📃";
    display: block;
    height: 100%;
    font-size: 24px;
    line-height: 1;
    text-indent: 0;
  }
  .uploader-file-icon[icon="folder"]::before {
    content: "📂";
  }
  .uploader-file-icon[icon="image"]::before {
    content: "📊";
  }
  .uploader-file-icon[icon="video"]::before {
    content: "📹";
  }
  .uploader-file-icon[icon="audio"]::before {
    content: "🎵";
  }
  .uploader-file-icon[icon="document"]::before {
    content: "📋";
  }
  .uploader-file-size {
    width: 13%;
    text-indent: 10px;
  }
  .uploader-file-meta {
    width: 8%;
  }
  .uploader-file-status {
    width: 24%;
    text-indent: 20px;
  }
  .uploader-file-actions {
    width: 10%;
  }
  .uploader-file-actions > span, .ep-uploader-progress-action > span {
    display: none;
    float: left;
    width: 16px;
    height: 16px;
    margin-top: 16px;
    margin-right: 10px;
    cursor: pointer;
    background: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAABkCAYAAAD0ZHJ6AAAAIGNIUk0AAHolAACAgwAA+f8AAIDpAAB1MAAA6mAAADqYAAAXb5JfxUYAAAAJcEhZcwAACxMAAAsTAQCanBgAAARkSURBVGje7ZnfS1NRHMAH4ptPkvQSuAdBkCxD8FUQJMEULUgzy1KyyPVQ4JMiiP4Bvg6EwUQQfMmwhwRDshwaKUjDVCgoSdDNHkzTJZ6+Z37Purve8+PeTb2TM/ggu+ew89l33x8H9BBCPG7GowXTJej3+wnDvEm0JuLC04+EYWftVAUv+fiCvDUdQR1BHUEdQR3BTIygvixoQS14XgTtthLVdpNWwXRLqvQ724LplFRtyrYF0yVpFLQrKRVMh6RZ0I6kkmCqklaCqpKZH0FX56Crq9jVfdDVk0RfFrSgFsxkQVmLcdKCVrKySCrryhPEyYShhzOcrFtG0EoilfHHk1CRU5rF6ZjNZhlVOW6RnMSVyyilKies4pO41diVy8wIujoHXV3FGdMHXTtJKLFYTLhZtq4vC1rwXApCZTIqgR6g1PBMCO9DL3bMMSqBHqDU8EyISDAHiGKvWwcCQG2KgjlAFCDAOhAAap0K5gKLphk8mqJgLrCIgoxRJ4J5wKpJ7gAoMkn5EBXBPGDVJHcAFJmkfIhQcAql1oBpTvTol9gG9pm4RHAKpdaAaU706JfYBvaZuJVgPQrt4sFlnOh5MC/p3lmJYD0K7eLBZZzoeTAv6d5ZnuAYHjpgEOnk5F0ufhG6v1ggOIaHDhhEOjl5l4tfhO4vthLcwAMrFNvLJO5vEwhu4IEViu1lEve3WQmyoihQFBzG/V0CQVYUBYqCw7i/SxTBcpsRbFeIYLnNCLZbCY5b5KAnxRwct8hBj9McZFVMW0ihRNBuFdMWUigRlFaxuQ9WWYjRMTiIe5z0wSoLMToGB3GPsA9aTZIJoB+nRgBnM1tzOkkmgH6cGgGczWzNpzqLx3n/aULJJgezeNw07oxQySbVywKjBOgFRnDs+VEsx8FlgVEC9AIjOPb8KJYjvSzoG7UW1IJaUAtqQS14toLNM5fN5APdwBJA8G83Pk/aK/rgzVvXzeQD3cASQPBvNz5P2ssTzAaGUIrHEO6zI5gNDKEUjyHcxxWkh4Ylcowwk1QQpIeGJXKMMJO0EgwqyjGCioJBJvDrxRMSuVOTJEXfbz1/bHwWtBL0yoQehK6RucgE+bGzanzulQh6E3IgQV+xpc8kcrfuSO7eTfJ3ZYmQw0Oy9azVKOk1C/bJ5D5F38YPeLfx0rjWJxHsS0SqsSYuxySjj5qO5Oj7xQWy2VBtFOwzCy6ryH3YfE3uh64Y1xckgstJPydEjkkeHv07Iy4Xaao15+KCWTBx6M/db+T9xivSErqaJDdzXI6yLRE8Vgg0coex/SPJvT0SbWu0KpZtbgSpCH3NRt7I5OxHkObc6heU+/M/J5vrpBFM5GBLqCQux14COXs5CNXK5OjPGm1tSMrJSOMNYQ4mVTGV/L6zTL7+DovkbFUxbSW0Wo05l8hJWsU+cRWfSh+Mt5Lb1ck/J1TvVsdDaR/MiEni+llsdZuZp62EViu+96bpNjNPWwmtVnzvFd5m9IVVC54x/wA7gNvqFG9vXQAAAABJRU5ErkJggg==") no-repeat 0 0;
  }
  .ep-uploader-progress-action > span {
    margin-top: -4px;
  }

  .uploader-file-actions > span:hover, ep-uploader-progress-action > span:hover {
    background-position-x: -21px;
  }
  .uploader-file-actions .uploader-file-pause, .ep-uploader-progress-action .uploader-file-pause {
    background-position-y: 0;
  }
  .uploader-file-actions .uploader-file-resume, .ep-uploader-progress-action .uploader-file-resume {
    background-position-y: -17px;
  }
  .uploader-file-actions .uploader-file-retry, .ep-uploader-progress-action .uploader-file-retry {
    background-position-y: -53px;
  }
  .uploader-file-actions .uploader-file-remove, .ep-uploader-progress-action .uploader-file-remove {
    display: block;
    background-position-y: -34px;
  }
  /* 修改过后的样式 */
  .ep-uploader-file-list {
    padding: 4px;
    color: #515a6e;
    border-radius: 4px;
    transition: background-color .2s ease-in-out;
    overflow: hidden;
    position: relative;
    box-sizing: border-box;
    -webkit-tap-highlight-color: transparent;
  }
  .ep-uploader-progress {
    display: inline-block;
    width: 100%;
    font-size: 12px;
    position: relative;
  }
  .ep-uploader-progress-show-info .ep-uploader-progress-outer {
    padding-right: 110px;
    margin-right: -110px;
  }
  .ep-uploader-progress-outer {
    display: inline-block;
    width: 100%;
    margin-right: 0;
    padding-right: 0;
  }
  .ep-uploader-progress-inner {
    display: inline-block;
    width: 100%;
    background-color: #f3f3f3;
    border-radius: 100px;
    vertical-align: middle;
    position: relative;
  }
  .ep-uploader-progress-bg {
    text-align: right;
    border-radius: 100px;
    background-color: #2d8cf0;
    transition: all .2s linear;
    position: relative;
    height: 3px;
  }
  .ep-uploader-progress-bg:after, .ep-uploader-progress-success-bg:after, .ep-uploader-progress-error-bg:after {
    content: '';
    display: inline-block;
    height: 100%;
    vertical-align: middle;
  }
  .ep-uploader-progress-success-bg {
    border-radius: 100px;
    background-color: #19be6b;
    transition: all .2s linear;
    position: absolute;
    top: 0;
    left: 0;
    height: 3px;
  }
  .ep-uploader-progress-error-bg {
    border-radius: 100px;
    background-color: rgb(230, 93, 12);
    transition: all .2s linear;
    position: absolute;
    top: 0;
    left: 0;
    height: 3px;
  }
  .ep-uploader-progress-text {
    display: inline-block;
    margin-left: 5px;
    text-align: left;
    font-size: 1em;
    vertical-align: middle;
    color: #808695;
  }
  :after, :before {
    box-sizing: border-box;
  }
  .ep-uploader-file-list:hover {
    background: #f3f3f3;
  }
  .ep-uploader-file-list:hover>span {
    color: #2d8cf0;
  }
  .ep-uploader-file-list>span {
    cursor: pointer;
    transition: color .2s ease-in-out;
  }
  .ivu-upload-list-remove {
    opacity: 0;
    font-size: 18px;
    cursor: pointer;
    float: right;
    margin-right: 4px;
    color: #999;
    transition: all .2s ease;
  }
  .ivu-icon {
    display: inline-block;
    font-family: Ionicons;
    speak: none;
    font-style: normal;
    font-weight: 400;
    font-variant: normal;
    text-transform: none;
    text-rendering: optimizeLegibility;
    line-height: 1;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    vertical-align: -.125em;
    text-align: center;
  }
  .ep-uploader-file-name {
  }
  * {
    box-sizing: border-box;
    -webkit-tap-highlight-color: transparent;
  }
</style>
