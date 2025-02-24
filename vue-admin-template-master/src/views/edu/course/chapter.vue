<template>

  <div class="app-container">

    <h2 style="text-align: center;">发布新课程</h2>

    <el-steps :active="2" process-status="wait" align-center style="margin-bottom: 40px;">
      <el-step title="填写课程基本信息"/>
      <el-step title="创建课程大纲"/>
      <el-step title="最终发布"/>
    </el-steps>

    <el-button type="text" @click="openChapterDialog()">添加章节</el-button>

    <!-- //拟态框 -->
    <el-dialog :visible.sync="dialogChapterFormVisible" title="添加章节">
      <el-form :model="chapter" label-width="120px">
        <el-form-item label="章节标题">
          <el-input v-model="chapter.title"/>
        </el-form-item>
        <el-form-item label="章节排序">
          <el-input-number v-model="chapter.sort" :min="0" controls-position="right"/>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogChapterFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveOrUpdate()">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 添加和修改课时表单 -->
    <el-dialog :visible.sync="dialogVideoFormVisible" title="添加课时">
      <el-form :model="video" label-width="120px">
        <el-form-item label="课时标题">
          <el-input v-model="video.title"/>
        </el-form-item>
        <el-form-item label="课时排序">
          <el-input-number v-model="video.sort" :min="0" controls-position="right"/>
        </el-form-item>
        <el-form-item label="是否免费">
          <el-radio-group v-model="video.isFree">
            <el-radio :label="0">免费</el-radio>
            <el-radio :label="1">收费</el-radio>
          </el-radio-group>
        </el-form-item>

        <el-form-item label="上传视频">
          <el-upload
            :on-success="handleVodUploadSuccess"
            :on-remove="handleVodRemove"
            :before-remove="beforeVodRemove"
            :on-exceed="handleUploadExceed"
            :file-list="fileList"
            :action="BASE_API+'/eduvod/video/uploadAliyunVideo'"
            :limit="1"
            class="upload-demo">
            <el-button size="small" type="primary">上传视频</el-button>
            <el-tooltip placement="right-end">
              <div slot="content">最大支持1G，<br>
                支持3GP、ASF、AVI、DAT、DV、FLV、F4V、<br>
                GIF、M2T、M4V、MJ2、MJPEG、MKV、MOV、MP4、<br>
                MPE、MPG、MPEG、MTS、OGG、QT、RM、RMVB、<br>
                SWF、TS、VOB、WMV、WEBM 等视频格式上传</div>
              <i class="el-icon-question"/>
            </el-tooltip>
          </el-upload>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVideoFormVisible = false">取 消</el-button>
        <el-button :disabled="saveVideoBtnDisabled" type="primary" @click="saveOrUpdateVideo">确 定</el-button>
      </div>
    </el-dialog>

    <ul class="chanpterList">
      <li
        v-for="chapter in chapterVideoList"
        :key="chapter.id">
        <p>
          {{ chapter.title }}
          <span class="acts">
            <el-button type="text" @click="openVideo(chapter.id)">添加小节</el-button>
            <el-button type="text" @click="openEditChapter(chapter.id)">编辑</el-button>
            <el-button type="text" @click="deleteChapter(chapter.id)">删除</el-button>
          </span>
        </p>

        <!-- 视频 -->
        <ul class="chanpterList videoList">
          <li
            v-for="video in chapter.children"
            :key="video.id">
            <p>{{ video.title }}
              <span class="acts">
                <el-button type="text" @click="openEditVideo(video.id)">编辑</el-button>
                <el-button type="text" @click="removeVideo(video.id)">删除</el-button>
              </span>
            </p>
          </li>
        </ul>
      </li>
    </ul>

    <el-form label-width="120px">
      <el-form-item>
        <el-button @click="previous">上一步</el-button>
        <el-button :disabled="saveBtnDisabled" type="primary" @click="next">下一步</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>

import chapter from '@/api/edu/chapter'
import video from '@/api/edu/video'

export default {
  data() {
    return {
      chapterVideoList: [],
      courseId: '',
      saveBtnDisabled: false, // 保存按钮是否禁用
      dialogChapterFormVisible: false,
      dialogVideoFormVisible: false, //小节弹框
      video: {
        title: '',
        sort: 0,
        isFree: 0,
        videoSourceId: '',
        videoOriginalName:''
      },
      chapter: { // 封装章节数据
        title: '',
        sort: 0,
        courseId: ''
      },
      fileList: [],//上传文件列表
      BASE_API: process.env.BASE_API // 接口API地址
    }
  },

  created() {
    // 获取路由中的id
    if (this.$route.params && this.$route.params.id) {
      this.courseId = this.$route.params.id
      this.getAllChapterVideo()
    }
  },

  methods: {
    //小节操作===========================================================================================
    //点击确定删除按钮执行的方法
    handleVodRemove(){
      //调用删除接口
      video.deleteAlyVod(this.video.videoSourceId)
        .then(res => {
          this.$message({
            type: 'success',
            message: '删除视频成功!'
          })
          //文件列表清空
          this.fileList = []
          //需要把这两个清空，否则会加入数据库
          this.video.videoSourceId = ''
          this.video.videoOriginalName = ''
        })
    },
    //beforeVodRemove点×按钮
    beforeVodRemove(file,fileList){
      return this.$confirm(`确定删除${file.name}？`)
    },
    //上传视频成功方法
    handleVodUploadSuccess(response, file, fileList){
      this.video.videoSourceId = response.data.videoId
      this.video.videoOriginalName = file.name
    },
    //视频上传之前方法
    handleUploadExceed(){
      this.$message.warning('想要重新上传视频，请先删除已上传的视频')
    },
    //编辑操作
    openEditVideo(videoId){
      this.dialogVideoFormVisible = true
      video.getVideo(videoId)
        .then(res => {
          this.video = res.data.chapter
        })
    },
    updateVideo(){
      video.updateVideo(this.video)
        .then(res => {
          this.dialogVideoFormVisible = false
          this.$message({
            type: 'success',
            message: '修改小节成功!'
          })
          // 刷新页面
          this.getAllChapterVideo()
          this.video.isFree = ''
          this.video.sort = 0
          this.video.videoSourceId = ''
          this.video.title = ''
          this.video.id = ''
        })
    },
    //删除小节
    removeVideo(videoId) {
      this.$confirm('此操作将删除小节记录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        video.deleteVideo(videoId).then(
          res => {
            this.$message({
              type: 'success',
              message: '删除小节成功!'
            })
            // 刷新页面
            this.getAllChapterVideo()
          }
        )
      })
    },
    openVideo(chapterId) {
      //弹窗
      this.dialogVideoFormVisible = true
      //设置章节id
      this.video.chapterId = chapterId
    },

    //添加小节
    addVideo() {
      this.video.courseId = this.courseId // 设置课程id
      video.addVideo(this.video).then(
        res => {
          this.dialogVideoFormVisible = false //关闭弹框
          this.$message({
            type: 'success',
            message: '添加小节成功!'
          })
          // 刷新页面
          this.getAllChapterVideo()
          this.video.isFree = ''
          this.video.sort = 0
          this.video.videoSourceId = ''
          this.video.title = ''
          this.video.id = ''
          this.fileList = []
        }
      )
    },
    saveOrUpdateVideo() {
      if (!this.video.id) {
        this.addVideo()
      } else {
        this.updateVideo()
      }
    },

    //章节操作===========================================================================================

    // 删除章节
    deleteChapter(chapterId) {
      this.$confirm('此操作将删除章节记录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        chapter.deleteChapter(chapterId).then(
          res => {
            this.$message({
              type: 'success',
              message: '删除章节成功!'
            })
            // 刷新页面
            this.getAllChapterVideo()
          }
        )
      })
    },

    // 修改章节弹框数据回显
    openEditChapter(chapterId) {
      this.dialogChapterFormVisible = true
      chapter.getChapter(chapterId)
        .then(res => {
          this.chapter = res.data.chapter
        })
    },
    openChapterDialog() {
      // 弹框弹出
      this.dialogChapterFormVisible = true
      // 表单数据清空
      this.chapter.title = ''
      this.chapter.sort = 0
      this.chapter.id = ''
    },

    // 添加章节
    addChapter() {
      // 设置课程id
      this.chapter.courseId = this.courseId
      chapter.addChapter(this.chapter)
        .then(res => {
          this.dialogChapterFormVisible = false
          this.$message({
            type: 'success',
            message: '添加章节成功!'
          })
          // 刷新页面
          this.getAllChapterVideo()
        })
    },

    // 修改章节的方法
    updateChapter() {
      chapter.updateChapter(this.chapter)
        .then(res => {
          this.dialogChapterFormVisible = false
          this.$message({
            type: 'success',
            message: '修改章节成功!'
          })
          // 刷新页面
          this.getAllChapterVideo()
        })
    },
    saveOrUpdate() {
      if (!this.chapter.id) {
        this.addChapter()
      } else {
        this.updateChapter()
      }
    },

    // 根据id查章节小节
    getAllChapterVideo() {
      chapter.getAllChapterVideo(this.courseId).then(
        res => {
          this.chapterVideoList = res.data.allChapterVideo
        }
      )
    },
    previous() {
      this.$router.push({ path: '/course/info/' + this.courseId })
    },

    next() {
      console.log('next')
      this.$router.push({ path: '/course/publish/' + this.courseId })
    }
  }
}
</script>
<style scoped>
.chanpterList {
  position: relative;
  list-style: none;
  margin: 0;
  padding: 0;
}

.chanpterList li {
  position: relative;
}

.chanpterList p {
  position: relative;
  float: left;
  font-size: 20px;
  margin: 10px 0;
  padding: 10px;
  height: 70px;
  line-height: 50px;
  width: 100%;
  border: 1px solid #DDD;
}

.chanpterList .acts {
  position: relative;
  z-index: 1;
  float: right;
  font-size: 14px;
}

.videoList {
  padding-left: 50px;
}

.videoList p {
  float: left;
  font-size: 14px;
  margin: 10px 0;
  padding: 10px;
  height: 50px;
  line-height: 30px;
  width: 100%;
  border: 1px dotted #DDD;
}

</style>
