<template>

  <div class="app-container">

    <h2 style="text-align: center;">发布新课程</h2>

    <el-steps :active="1" process-status="wait" align-center style="margin-bottom: 40px;">
      <el-step title="填写课程基本信息"/>	
      <el-step title="创建课程大纲"/>
      <el-step title="最终发布"/>
    </el-steps>

    <el-form label-width="120px">

        <el-form-item label="课程标题">
            <el-input v-model="courseInfo.title" placeholder=" 示例：机器学习项目课：从基础到搭建项目视频课程。专业名称注意大小写"/>
        </el-form-item>

        <!-- 所属分类 TODO -->
        <!-- 所属分类：级联下拉列表 -->
        <!-- 一级分类 -->
        <el-form-item label="课程类别">
        <el-select
            v-model="courseInfo.subjectParentId"
            placeholder="一级分类" @change="subjectLevelOneChanged">
            <el-option
            v-for="subject in subjectOneList"
            :key="subject.id"
            :label="subject.title"
            :value="subject.id"/>
        </el-select>
        <!-- 二级分类 -->
        <el-select v-model="courseInfo.subjectId" placeholder="二级分类">
        <el-option
            v-for="subject in subjectTwoList"
            :key="subject.id"
            :label="subject.title"
            :value="subject.id"/>
        </el-select>
        </el-form-item>

        <!-- 课程讲师 TODO -->
        <!-- 课程讲师 -->
        <el-form-item label="课程讲师">
        <el-select
            v-model="courseInfo.teacherId"
            placeholder="请选择">
            <el-option
            v-for="teacher in teacherList"
            :key="teacher.id"
            :label="teacher.name"          
            :value="teacher.id"/>
        </el-select>
        </el-form-item>

        <el-form-item label="总课时">
            <el-input-number :min="0" v-model="courseInfo.lessonNum" controls-position="right" placeholder="请填写课程的总课时数"/>
        </el-form-item>

        <!-- 课程简介 TODO -->
        <!-- 课程简介-->
        <el-form-item label="课程简介">
            <tinymce :height="300" v-model="courseInfo.description"/>
        </el-form-item>

        <!-- 课程封面 TODO -->
        <!-- 课程封面-->
        <el-form-item label="课程封面">

        <el-upload
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
            :action="BASE_API+'/admin/oss/file/upload?host=cover'"
            class="avatar-uploader">
            <img :src="courseInfo.cover">
        </el-upload>

        </el-form-item>

        <el-form-item label="课程价格">
            <el-input-number :min="0" v-model="courseInfo.price" controls-position="right" placeholder="免费课程请设置为0元"/> 元
        </el-form-item>

        <el-form-item>
            <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate">保存并下一步</el-button>
        </el-form-item>
    </el-form>
  </div>
</template>
<script>
import course from '@/api/edu/course'
import subject from '@/api/edu/subject'
import Tinymce from '@/components/Tinymce'      // 引入组件

const defaultForm = {
        title: '',  
        subjectId: '',      // 二级分类id
        subjectParentId:'', // 一级分类id
        teacherId: '',
        lessonNum: 0,
        description: '',
        cover: '/static/01.png',
        price: 0
}

export default {
    // 声明组件
    components: { Tinymce },
    data() {
        return {
            saveBtnDisabled: false, // 保存按钮是否禁用
            courseInfo:defaultForm,
            teacherList: [],
            subjectOneList: [],      // 所有的一级分类
            subjectTwoList: []      // 所有的二级分类
        }
    },
    created(){
        // 初始化所有讲师
        this.getListTeacher()
        // 初始话一级分类
        this.getOneList()
    },
    methods:{
        // 封面上传成功之后做的事情【就是进行显示】
        handleAvatarSuccess(res, file) {
            this.courseInfo.cover = res.data.url
        },
        // 上传之前需要做的事情【规定上传什么格式的文件】
        beforeAvatarUpload(file) {
            const isJPG = file.type === 'image/jpeg'        // 规定是图片格式
            const isLt2M = file.size / 1024 / 1024 < 2      // 上传大小不超过2M 

            if (!isJPG) {
                this.$message.error('上传头像图片只能是 JPG 格式!')
            }
            if (!isLt2M) {
                this.$message.error('上传头像图片大小不能超过 2MB!')
            }
            return isJPG && isLt2M
        },
        // 点击某个一级事件的时候，触发对应的二级分类
        subjectLevelOneChanged(value) {
           // alert(value)
           // 1. 首先遍历所有的一级和二级
           for (var i = 0; i < this.subjectOneList.length; i++) {
               var oneSubject = this.subjectOneList[i]
               // 判断所有的一级的id和我们点击的是不是一样
               if (value === oneSubject.id) {
                   // c从一级中取二级
                   this.subjectTwoList = oneSubject.children
                   // 将二级的清空
                   this.courseInfo.subjectId = ''
               }
           }
        },
        // 查询所有的一级分类
        getOneList() {
            subject.getSubjectList()
                .then(response => {
                    this.subjectOneList = response.data.list
                })
        },
        getListTeacher() {
            course.getListTeacher()
                .then(response => {
                    this.teacherList = response.data.items
                })
        },
        saveOrUpdate() {
            course.addCourseInfo(this.courseInfo)
                .then(response => {
                    // 提示信息
                    this.$message({
                        type: 'success',
                        message: '添加课程信息成功!'
                    });
                    // 添加章节和小结部分，跳转到第二部
                    this.$router.push({path:'/course/chapter/'+(response.data.courseId)})
                })
            
        }
    }
}
</script>
<style scoped>
.tinymce-container {
  line-height: 29px;
}
</style>s