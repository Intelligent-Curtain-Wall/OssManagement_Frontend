<template>
  <div class='data-pass'>
    <el-tabs type='border-card'>
      <el-tab-pane label='文件传输控制台'>
        <div class='tabs-item'>
          <div class='control'>
            <div class='upload-btns'>
              <el-tooltip class='item' effect='dark' content='启动全部' placement='top'>
                <div class='btn-child' @click="fileControl('start')">
                  <span class='iconfont icon-ai23'/>
                </div>
              </el-tooltip>
              <el-tooltip class='item' effect='dark' content='暂停全部' placement='top'>
                <div class='btn-child' @click="fileControl('stop')">
                  <span class='iconfont icon-pause'/>
                </div>
              </el-tooltip>
              <el-tooltip class='item' effect='dark' content='清空已完成' placement='top'>
                <div class='btn-child' @click="emptyFile('has')">
                  <span class='iconfont icon-lajitong'/>
                </div>
              </el-tooltip>
              <el-tooltip class='item' effect='dark' content='清空全部' placement='top'>
                <div class='btn-child' @click="emptyFile('all')">
                  <span class='iconfont icon-lajitong'/>
                </div>
              </el-tooltip>
            </div>
          </div>
          <div class='upload-list'>
            <div class='upload-item' v-for='(item, index) in uploadList' :key='index'>
              <div class='upload-info'>
                <span>{{ item.title }}</span>
                <div class='progress'>
                  <div class='progress-child'
                       :style="{ width: numberTwoDecimal(item.progress * 100) + '%', backgroundColor: item.status === '0' ? '#1E90FF' : '#FFD700' }"/>
                </div>
              </div>
              <span class='upload-status'>
                {{ item.status === '0' ? '上传中' : item.status === '-1' ? '暂停' : '完成' }}
              </span>
              <span class='upload-size'>
                {{ item.currentSize > item.size ? renderSize(item.size) : renderSize(item.currentSize) }} /
                {{ renderSize(item.size) }}
              </span>
              <div class='upload-btns'>
                <div class='btn-child' @click='stopClick(item)' v-if="item.status !== '1'">
                  <span :class="`iconfont ${ item.status === '0' ? 'icon-pause' : 'icon-ai23' }`"/>
                </div>
                <div class='btn-child' @click="emptyFile('key', item.uploadId)">
                  <span class='iconfont icon-dashujukeshihuaico-'/>
                </div>
              </div>
            </div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
    <span class='iconfont icon-dashujukeshihuaico-'
          @click="$store.commit('stateUpdate', { name: 'transmissionShow', data: false })"/>
  </div>
</template>

<script lang='js'>
import {mapState} from 'vuex'
import {renderSize, numberTwoDecimal} from '@/tool'

export default {
  data() {
    return {
      renderSize,
      numberTwoDecimal: numberTwoDecimal,
      client: null
    }
  },
  computed: mapState({
    uploadList: (state) => state.uploadList
  }),
  mounted() {
    this.emptyFile('not')
  },
  methods: {
    /**
     * Handles the click event for the pause/start upload button.
     * Based on the current status of the file, either pauses or resumes the upload.
     * If the file is paused, it triggers the Vuex action to resume the upload;
     * If the file is uploading, it cancels the upload.
     * @param {Object} item - The current upload file object.
     */
    stopClick(item) {
      if (item.status === '-1') {
        this.$store.dispatch('sliceUpload', item)
      } else {
        item.client.cancel()
      }
    },

    /**
     * Controls the upload state for all files (start/pause).
     * Based on the provided control type ('start' or 'stop'), it either starts or pauses the upload for all files.
     *
     * @param {String} type - Control type, 'start' to start uploads, 'stop' to pause uploads.
     */
    fileControl(type) {
      let uploadList = this.uploadList
      if (type === 'start') {
        uploadList.map((item) => {
          if (item.status === '-1') {
            this.$store.dispatch('sliceUpload', item)
          }
        })
      } else {
        uploadList.map((item) => {
          if (item.status === '0') {
            item.client.cancel()
          }
        })
      }
    },

    /**
     * Clears files from the upload list based on the provided type.
     * Supports deleting completed, ongoing, or all files.
     *
     * @param {String} type - The type of operation ('all' for clearing all, 'not' for completed files, 'has' for ongoing uploads, 'key' for deleting a specific file).
     * @param {String} [uploadId] - The uploadId of the file to be deleted (only required if type is 'key').
     */
    emptyFile(type, uploadId) {
      let uploadList = JSON.parse(window.localStorage.getItem('uploadList')) || []
      if (type === 'all') {
        uploadList = []
      } else if (type === 'not') {
        uploadList = uploadList.filter((item) => item.progress === 1)
      } else if (type === 'has') {
        uploadList = uploadList.filter((item) => item.progress < 1)
      } else if (type === 'key') {
        uploadList = uploadList.filter((item) => item.uploadId !== uploadId)
      }
      window.localStorage.setItem('uploadList', JSON.stringify(uploadList))
      this.$store.commit('stateUpdate', {
        name: 'uploadList',
        data: uploadList
      })
    }
  }
}
</script>

<style lang='css'>
/* noinspection CssUnusedSymbol */
.el-tabs--border-card > .el-tabs__content {
  padding: 0;
}
</style>

<style scoped lang='css'>
.data-pass {
  width: 600px;
  position: absolute;
  right: 0;
  bottom: 0;
  opacity: 0.9;
  z-index: 999;
  user-select: none;
}

.data-pass .tabs-item .control {
  padding-left: 12px;
  border-bottom: 1px solid rgba(136, 136, 136, 0.3);
}

.data-pass .tabs-item .upload-list {
  height: 360px;
  overflow: auto;
}

.data-pass .tabs-item .upload-list .upload-item {
  height: 50px;
  border-bottom: 1px solid rgba(136, 136, 136, 0.3);
  padding-left: 14px;
  display: flex;
  align-items: center;
  position: relative;
}

.data-pass .tabs-item .upload-list .upload-item .upload-info {
  width: 220px;
}

.data-pass .tabs-item .upload-list .upload-item .upload-info .progress {
  width: 200px;
  height: 6px;
  border-radius: 3px;
  margin-top: 5px;
  border: 1px solid rgba(136, 136, 136, 0.3);
}

.data-pass .tabs-item .upload-list .upload-item .upload-info .progress .progress-child {
  height: 100%;
  background-color: #5182F8;
}

.data-pass .tabs-item .upload-list .upload-item .upload-status {
  width: 100px;
  text-align: center;
  color: #337AB7;
  font-size: 13px;
  font-weight: 700;
}

.data-pass .tabs-item .upload-list .upload-item .upload-size {
  font-size: 13px;
  color: #2F3235;
  margin-left: 20px;
}

.data-pass .tabs-item .upload-list .upload-item .upload-btns {
  height: 100%;
  position: absolute;
  right: 14px;
}

.data-pass .tabs-item .upload-list .upload-item .upload-btns .btn-child {
  width: 20px;
  height: 20px;
}

.data-pass > .icon-dashujukeshihuaico- {
  position: absolute;
  right: 7px;
  top: 7px;
  font-size: 24px;
  cursor: pointer;
}

.upload-btns {
  height: 40px;
  display: flex;
  align-items: center;
}

.upload-btns .btn-child {
  width: 40px;
  height: 30px;
  border: 1px solid rgba(136, 136, 136, 0.3);
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  margin-right: 5px;
  border-radius: 5px;
}

.upload-btns .btn-child:hover {
  background-color: #E6E6E6;
}

.upload-btns .btn-child:active {
  background-color: #F5F5F5;
}
</style>
