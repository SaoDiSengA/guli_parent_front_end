<template>
  <div class="app-container">
    <el-form label-width="120px">
      <el-form-item label="讲师名称">
        <el-input v-model="teacher.name"/>
      </el-form-item>
      <el-form-item label="讲师排序">
        <el-input-number v-model="teacher.sort" controls-position="right" :min="0"/>
      </el-form-item>
      <el-form-item label="讲师头衔">
        <el-select v-model="teacher.level" clearable placeholder="请选择">
          <!--
            数据类型一定要和取出的json中的一致，否则没法回填
            因此，这里value使用动态绑定的值，保证其数据类型是number
          -->
          <el-option :value="1" label="高级讲师"/>
          <el-option :value="2" label="首席讲师"/>
        </el-select>
      </el-form-item>
      <el-form-item label="讲师资历">
        <el-input v-model="teacher.career"/>
      </el-form-item>
      <el-form-item label="讲师简介">
        <el-input v-model="teacher.intro" :rows="10" type="textarea"/>
      </el-form-item>

      <!-- 讲师头像：TODO -->
      <!-- 讲师头像 -->
      <el-form-item label="讲师头像">

        <!-- 头衔缩略图 -->
        <pan-thumb :image="teacher.avatar"/>
        <!-- 文件上传按钮 -->
        <el-button type="primary" icon="el-icon-upload" @click="imagecropperShow=true">更换头像</el-button>
                <!--
            v-show：是否显示上传组件
            :key：类似于id，如果一个页面多个图片上传控件，可以做区分
            :url：后台上传的url地址
            @close：关闭上传组件
            @crop-upload-success：上传成功后的回调 -->
        <image-cropper
            v-show="imagecropperShow"
            :width="300"
            :height="300"
            :key="imagecropperKey"
            :url="BASE_API+'/eduoss/fileoss'"
            field="file"
            @close="close"
            @crop-upload-success="cropSuccess"/>
      </el-form-item>

      <el-form-item>
        <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate()">保存</el-button>
      </el-form-item>
      
    </el-form>
  </div>
</template>

<script>
import teacherApi from '@/api/edu/teacher'
import ImageCropper from '@/components/ImageCropper'
import PanThumb from '@/components/PanThumb'

export default {
  components:{ImageCropper,PanThumb},
  data(){
    return{
      
      teacher:{
        name:'',
        sort:0,
        level:1,
        career:'',
        intro:'',
        avatar:''
      },
      //上传弹框是否显示
      imagecropperShow:false,
      imagecropperKey:0,
      BASE_API:process.env.BASE_API,//获取dev.env.js
      saveBtnDisabled:false //保存按钮是否禁用，多次提交问题
    }
  },
  created(){
    this.init()
  // if(this.$route.params && this.$route.params.id){
  //   let id = this.$route.params.id
  //   this.getTeacherInfo(id)
  // }
  // if(this.$route.params.id == ''){
  //   this.teacher = {}
  // }    不起作用
  },
  watch:{
    $route(to,from){
      this.init()
    }
  },
  methods:{
    cropSuccess(data){
      //上传之后的接口返回图片的地址
      this.teacher.avatar = data.url
      this.imagecropperShow = false
      this.imagecropperKey = this.imagecropperKey + 1
    },//上传成功
    close(){
      this.imagecropperShow = false
      //上传组件初始化
      this.imagecropperKey = this.imagecropperKey + 1
    },//关闭上传弹窗
    init(){
        if(this.$route.params && this.$route.params.id){
          let id = this.$route.params.id
          this.getTeacherInfo(id)
        }else{
          this.teacher = {}
        }
    },
    saveOrUpdate(){
      //判断是修改还是添加
      //根据teacher中是否有id
      if(this.teacher.id){
        this.editTeacher()
      }else{
        this.saveTaecher()
      }
    },
    //修改
    editTeacher(){
      teacherApi.editTeacherInfo(this.teacher).then(res => {
        this.$message({
            type: 'success',
            message: '修改成功!'
          });
          //回到列表界面 路由跳转
          this.$router.push({path:'/teacher/teacherList'})
      })
    },
    //添加讲师方法
    saveTaecher(){
      teacherApi.addTeacher(this.teacher)
        .then(res => {
          //提示成功
          this.$message({
            type: 'success',
            message: '添加成功!'
          });
          //回到列表界面 路由跳转
          this.$router.push({path:'/teacher/teacherList'})
        })
    },
    getTeacherInfo(id){
      teacherApi.getTeacherInfo(id)
        .then(res => {
          this.teacher = res.data.teacher
        })
    }
  }
}
</script>