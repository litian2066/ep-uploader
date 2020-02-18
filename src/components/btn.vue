<template>
  <label class="uploader-btn" ref="btn" v-show="support">
    <slot></slot>
  </label>
</template>

<script>
  import { uploaderMixin, supportMixin } from '../common/mixins'

  const COMPONENT_NAME = 'uploader-btn'

  export default {
    name: COMPONENT_NAME,
    mixins: [uploaderMixin, supportMixin],
    props: {
      directory: {
        type: Boolean,
        default: false
      },
      single: {
        type: Boolean,
        default: false
      },
      attrs: {
        type: Object,
        default () {
          return {}
        }
      }
    },
    mounted () {
      this.$nextTick(() => {
        this.uploader.uploader.assignBrowse(this.$refs.btn, this.directory, this.single, this.attrs)
      })
    }
  }
</script>

<style>
  .uploader-btn {
    display: inline-block;
    position: relative;
    padding: 4px 8px;
    line-height: 1.4;
    color: #fff;
    border: 1px solid #666;
    cursor: pointer;
    background: #1890ff;
    border-color: #1890ff;
    outline: none;
    font-size: 14px;
    width: 90px;
    min-width: 60px;
    min-height: 26px;
    text-align: center;
    border-radius: 4px;
  }
  .uploader-btn:hover {
    border-color: #57a3f3;
  }
</style>
