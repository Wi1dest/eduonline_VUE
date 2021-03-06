<template>
  <div class="app-container">
    <h2 style="text-align: center;">发布课程大纲</h2>

    <el-steps :active="2" process-status="wait" align-center style="margin-bottom: 40px;">
      <el-step title="填写课程基本信息" />
      <el-step title="创建课程大纲" />
      <el-step title="最终发布" />
    </el-steps>

    <el-button type="text" @click="opendChapterFormVisible()">添加章节</el-button>

    <!-- 章节 -->
    <ul class="chanpterList">
      <li v-for="chapter in chapterVideoList" :key="chapter.id">
        <p>
          {{ chapter.title }}
          <span class="acts">
            <el-button style type="text" @click="openAddVideoFrom(chapter.id)">添加小节</el-button>
            <el-button style type="text" @click="editChapter(chapter.id)">编辑</el-button>
            <el-button type="text" @click="deleteChapter(chapter.id)">删除</el-button>
          </span>
        </p>

        <!-- 小节 -->
        <ul class="chanpterList videoList">
          <li v-for="video in chapter.children" :key="video.id">
            <p>
              {{ video.title }}
              <span class="acts">
                <el-button type="text" @click="editVideo(video.id)">编辑</el-button>
                <el-button type="text" @click="deleteVideo(video.id)">删除</el-button>
              </span>
            </p>
          </li>
        </ul>
      </li>
    </ul>
    <div>
      <el-button @click="previous">上一步</el-button>
      <el-button :disabled="saveBtnDisabled" type="primary" @click="next">下一步</el-button>
    </div>

    <!-- 添加和修改章节表单 -->
    <el-dialog :visible.sync="dialogChapterFormVisible" title="添加章节">
      <el-form :model="chapter" label-width="120px">
        <el-form-item label="章节标题">
          <el-input v-model="chapter.title" />
        </el-form-item>
        <el-form-item label="章节排序">
          <el-input-number v-model="chapter.sort" :min="0" controls-position="right" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogChapterFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveOrUpdate">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 添加和修改课时表单 -->
    <el-dialog :visible.sync="dialogVideoFormVisible" title="添加课时">
      <el-form :model="video" label-width="120px">
        <el-form-item label="课时标题">
          <el-input v-model="video.title" />
        </el-form-item>
        <el-form-item label="课时排序">
          <el-input-number v-model="video.sort" :min="0" controls-position="right" />
        </el-form-item>
        <el-form-item label="是否免费">
          <el-radio-group v-model="video.isFree">
            <el-radio :label="true">免费</el-radio>
            <el-radio :label="false">默认</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="上传视频">
          <el-upload
            :on-success="handleVodUploadSuccess"
            :on-remove="handleVodRemove"
            :before-remove="beforeVodRemove"
            :on-exceed="handleUploadExceed"
            :file-list="fileList"
            :action="BASE_API+'/vodservice/video/uploadAliVideo'"
            :limit="1"
            class="upload-demo"
          >
            <el-button size="small" type="primary">上传视频</el-button>
            <el-tooltip placement="right-end">
              <div slot="content">
                最大支持1G，
                <br />支持3GP、ASF、AVI、DAT、DV、FLV、F4V、
                <br />GIF、M2T、M4V、MJ2、MJPEG、MKV、MOV、MP4、
                <br />MPE、MPG、MPEG、MTS、OGG、QT、RM、RMVB、
                <br />SWF、TS、VOB、WMV、WEBM 等视频格式上传
              </div>
              <i class="el-icon-question" />
            </el-tooltip>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVideoFormVisible = false">取 消</el-button>
        <el-button :disabled="saveVideoBtnDisabled" type="primary" @click="saveOrUpdateVideo">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import chapter from "@/api/edu/chapter";
import video from "@/api/edu/video";

export default {
  data() {
    return {
      saveBtnDisabled: false, // 保存按钮是否禁用
      courseId: "", // 课程ID
      chapterVideoList: [], // 章节List
      chapter: {
        // 章节对象
        title: "",
        sort: 0,
        courseId: this.$route.params.id,
      },
      video: {
        // 课时对象
        chapterId: "",
        courseId: this.$route.params.id,
        title: "",
        sort: 0,
        isFree: 0,
        videoSourceId: "",
        videoOriginalName: "",
      },
      dialogChapterFormVisible: false, //是否显示章节表单
      dialogVideoFormVisible: false, //是否显示小节表单
      fileList: [], //上传文件列表
      BASE_API: process.env.BASE_API, // 接口API地址
    };
  },

  created() {
    //获取路由的id值
    if (this.$route.params && this.$route.params.id) {
      this.courseId = this.$route.params.id;
      //根据课程id查询章节和小节
      this.getChapterVideo();
    }
  },

  methods: {
    // 章节部分
    opendChapterFormVisible() {
      this.chapter.title = " ";
      this.chapter.sort = 0;
      this.dialogChapterFormVisible = true;
    },
    saveOrUpdate() {
      if (!this.chapter.id) {
        // 没有courseId说明是新增
        chapter.addChapter(this.chapter).then((response) => {
          this.dialogChapterFormVisible = false; //是否显示章节表单
          this.getChapterVideo();
          // 提示
          this.$message({
            type: "success",
            message: "添加章节成功!",
          });
        });
      } else {
        chapter.updateChapterByChapterId(this.chapter).then((response) => {
          this.dialogChapterFormVisible = false; //是否显示章节表单
          this.getChapterVideo();
          // 提示
          this.$message({
            type: "success",
            message: "修改章节成功!",
          });
        });
      }
    },
    deleteChapter(chapterId) {
      this.$confirm("此操作将永久删除章节记录及其全部小节, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        chapter.deleteChapterByChapterId(chapterId).then((response) => {
          //提示信息
          this.$message({
            type: "success",
            message: "删除成功!",
          });
          this.getChapterVideo();
        });
      });
    },
    getChapterVideo() {
      chapter.getAllChapterVideo(this.courseId).then((response) => {
        this.chapterVideoList = response.data;
      });
    },
    editChapter(chapterId) {
      this.dialogChapterFormVisible = true;
      chapter.getChapterByChapterId(chapterId).then((response) => {
        this.chapter = response.data;
      });
    },
    // 小节部分
    openAddVideoFrom(chapterId) {
      this.video.chapterId = chapterId;
      this.video.title = "";
      this.video.sort = 0;
      this.video.free = 0;
      this.video.videoOriginalName = "";
      this.video.videoSourceId = "";
      // this.videoSourceId = 0
      this.dialogVideoFormVisible = true;
    },
    saveOrUpdateVideo() {
      if (!this.video.id) {
        this.$confirm("请确保视频已经上传成功!(显示绿色√)", "提示", {
          confirmButtonText: "我已确定上传成功",
          cancelButtonText: "还没,再等等!",
          type: "warning",
        }).then(() => {
          video.addVideo(this.video).then((response) => {
            this.dialogVideoFormVisible = false; //是否显示小节表单
            this.getChapterVideo();
            // 提示
            this.$message({
              type: "success",
              message: "添加小节成功!",
            });
            this.fileList = [];
          });
        });
      } else {
        this.$confirm("请确保视频已经上传成功!(显示绿色√)", "提示", {
          confirmButtonText: "我已确定上传成功",
          cancelButtonText: "还没,再等等!",
          type: "warning",
        }).then(() => {
          video.updateVideo(this.video).then((response) => {
            this.dialogVideoFormVisible = false; //是否显示小节表单
            this.getChapterVideo();
            // 提示
            this.$message({
              type: "success",
              message: "修改小节成功!",
            });
            this.fileList = [];
          });
        });
      }
    },
    deleteVideo(videoId) {
      this.$confirm("此操作将永久删除该小节, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        video.deleteVideoByVideoId(videoId).then((response) => {
          //提示信息
          this.$message({
            type: "success",
            message: "删除成功!",
          });
          this.getChapterVideo();
        });
      });
    },
    editVideo(videoId) {
      this.dialogVideoFormVisible = true;
      video.getVideo(videoId).then((response) => {
        this.video = response.data;
      });
    },
    // 视频上传
    //成功回调
    handleVodUploadSuccess(response, file, fileList) {
      this.video.videoSourceId = response.data;
      this.video.videoOriginalName = file.name;
    },
    //视图上传多于一个视频
    handleUploadExceed(files, fileList) {
      this.$message.warning("想要重新上传视频，请先删除已上传的视频");
    },
    beforeVodRemove(file, fileList) {
      return this.$confirm(`确定删除 ${file.name}？`);
    },
    handleVodRemove(file, fileList) {
      video.deleeteVideo(this.video.videoSourceId).then((response) => {
        this.$message({
          type: "success",
          message: "删除视频成功",
        });
      });
    },
    // 通用部分
    previous() {
      //回到上一步
      this.$router.push({ path: "/course/info/" + this.courseId });
    },
    next() {
      //调到第三步
      this.$router.push({ path: "/course/publish/" + this.courseId });
    },
  },
};
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
  float: left;
  font-size: 20px;
  margin: 10px 0;
  padding: 10px;
  height: 70px;
  line-height: 50px;
  width: 100%;
  border: 1px solid #ddd;
}
.chanpterList .acts {
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
  border: 1px dotted #ddd;
}
</style>