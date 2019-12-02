<template>
  <d2-container class="page" type="card">
    <!-- <d2-page-cover>
      <d2-icon-svg class="logo" name="d2-admin"/>
      <template slot="footer">
        <div class="btn-group">
          <span class="btn-group__btn" @click="$open('https://github.com/d2-projects')">开源组织</span> |
          <span class="btn-group__btn" @click="$open('https://fairyever.com/d2-admin/doc/zh/')">文档</span> |
          <span class="btn-group__btn" @click="$open('https://github.com/d2-projects/d2-admin-start-kit')">简化版</span> |
          <span class="btn-group__btn" @click="$open('https://juejin.im/user/57a48b632e958a006691b946/posts')">掘金</span> |
          <span class="btn-group__btn" @click="$open('https://daily.fairyever.com')">日报</span> |
          <el-popover :width="172" trigger="hover">
            <p class="d2-mt-0 d2-mb-10">D2Projects</p>
            <img src="./image/qr@2x.png" style="width: 172px;">
            <span slot="reference" class="btn-group__btn btn-group__btn--link">
              <d2-icon name="weixin"/>
              微信公众号
            </span>
            <p style="font-size: 12px; margin-top: 0px; margin-bottom: 0px;">
              官方公众号，主要推送前端技术类文章、框架资源、学习教程，以及 D2 系列项目更新信息
            </p>
          </el-popover>
        </div>
        <d2-badge/>
        <d2-help-btn/>
      </template>
    </d2-page-cover>-->
    <el-row :gutter="20">
      <el-col :span="10">
        <el-card class="box-card" style="min-height:400px;">
          <div slot="header" class="clearfix">
            <span>上传车辆照片</span>
          </div>
          <el-form ref="form" label-width="80px">
            <el-upload
              class="upload-demo"
              ref="upload"
              action="apii/detect/"
              :multiple="false"
              :limit=1
              :on-change="handleChangeFile"
              :on-preview="handlePictureCardPreview"
              :on-remove="handleRemove"
              :file-list="fileList"
              list-type="picture-card"
              :auto-upload="false">
              <i class="el-icon-plus"></i>
              <!-- <el-button slot="trigger" size="small" type="primary">选取图片</el-button> -->
              <!-- <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">检测</el-button> -->
              <div slot="tip" class="el-upload__tip">支持多目标检测，可上传包含多个车辆的jpg/png/jpeg照片</div>
            </el-upload>
            <el-dialog :visible.sync="dialogVisible">
              <img width="100%" :src="dialogImageUrl" alt="">
            </el-dialog>
            <el-form-item>
              <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">检测</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
      <el-col :span="14">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>识别结果</span>
          </div>
          <d2-crud
            ref="d2Crud"
            :columns="columns"
            :data="data"/>
        </el-card>
        
      </el-col>
    </el-row>
    <div class="row">
      <div class="col-md-3">
        <div class="page-header">
          <h2>请上传车辆照片</h2>
          <h3>
            <small>支持多目标检测，可上传包含多个车辆的照片</small>
          </h3>
        </div>
        <form id="upload-file" method="post" enctype="multipart/form-data">
          <label for="imageUpload" class="upload-label">选择照片...</label>
          <input
            type="file"
            name="file"
            id="imageUpload"
            accept=".png, .jpg, .jpeg"
            @change="onUploadChange"
          />
        </form>

        <div v-if="originImgUrl">
          <img :src="originImgUrl" style="max-width: 200px" />
          <br />
          <br />

          <button
            type="button"
            :disabled="loading"
            class="btn btn-primary btn-lg"
            @click="onDetectBtnClick"
            id="btn-predict"
          >
            检测
            <div class="loader" v-if="loading"></div>
          </button>
        </div>
      </div>

      <div class="col-md-9">
        <div class="page-header">
          <h3>识别结果</h3>
        </div>
        <div class="alert alert-danger alert-dismissible fade in" role="alert" v-show="npNotFound">
          <h4>未识别到车牌信息!</h4>
          <p>请选择清晰度较高的照片, 车辆的车牌需要完整, 肉眼可识别。</p>
        </div>

        <div v-if="result">
          <table class="table table-hover table-condensed">
            <tr>
              <td>编号</td>
              <td>类型</td>
              <td>车牌裁剪前</td>
              <td>车牌裁剪后</td>
              <td>颜色</td>
              <td>车牌号</td>
            </tr>

            <tr v-for="(item,index) of result.lplist">
              <td>{{ index+1 }}</td>
              <td>汽车</td>
              <td>
                <img :src="item.seg" style="max-width: 100px" />
                <span style="color: red" v-if="item.isOrigin">*</span>
              </td>
              <td>
                <img :src="item.segModify" style="max-width: 100px" />
              </td>
              <td>{{ item.color }}</td>
              <td>{{ item.str }}</td>
            </tr>
          </table>
          <img :src="result.imgout" style="max-width: 600px" />
          <br />
        </div>
      </div>
    </div>
    <template slot="footer">
      <h4>
        复旦大学-1904深度学习小组作业-多目标车牌识别
        <small
          style="margin-left: 20px;"
        >小组成员(按姓氏顺序)：程竹君, 金月, 石荣伟, 唐士卫, 赵茂祥</small>
      </h4>
    </template>
  </d2-container>
</template>

<script>
// import D2HelpBtn from './components/d2-help-btn'
// import D2Badge from './components/d2-badge'
// import D2PageCover from './components/d2-page-cover'
import D2Crud from '@d2-projects/d2-crud'
import request from '@/plugin/axios'
export default {
  components: {
    D2Crud
    // D2HelpBtn,
    // D2Badge,
    // D2PageCover
  },
  data() {
    return {
      // filename: __filename,
      originImgUrl: "",
      loading: false,
      result: {},
      npNotFound: false,
      imageUrl: '',
      dialogVisible: false,
      detectData: {
        name: 'fudan',
        password: 'fudan'
      },
      detectHeaders: {
        'Authorization': 'Basic ZnVkYW46ZnVkYW4='
      },
      // fileList: [{name: 'food.jpeg', url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'}, {name: 'food2.jpeg', url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'}],
      fileList: [],
      fileObj: {},
      columns: [
        {
          title: '日期',
          key: 'date',
          width: '180'
        },
        {
          title: '姓名',
          key: 'name',
          width: '180'
        },
        {
          title: '地址',
          key: 'address'
        }
      ],
      data: [
        {
          date: '2016-05-02',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        },
        {
          date: '2016-05-04',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1517 弄'
        },
        {
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1519 弄'
        },
        {
          date: '2016-05-03',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1516 弄'
        }
      ],
      form: {}
    }
  },
  methods: {
    submitUpload() {
      // console.log(this.$refs.upload.fileList)
      this.$refs.upload.submit()
      // request({
      //   url: '/detect/',
      //   method: 'post',
      //   data: this.fileObj,
      //   headers: {
      //     'Content-Type': 'multipart/form-data',
      //     'Authorization': 'Basic ZnVkYW46ZnVkYW4=',
      //     'X-Requested-With': 'XMLHttpRequest'
      //   }
      // }).then(res => {
      //   console.log(res)
      // })
    },
    handleRemove(file, fileList) {
      console.log(file, fileList)
    },
    handlePreview(file) {
      console.log(file)
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    },
    handleChangeFile (file, fileList) {
      this.fileObj = file
    }
  }
}
</script>

<style lang="scss" scoped>
.page {
  .logo {
    width: 120px;
  }
  .btn-group {
    color: $color-text-placehoder;
    font-size: 12px;
    margin-top: 0px;
    margin-bottom: 20px;
    .btn-group__btn {
      color: $color-text-sub;
      &:hover {
        color: $color-text-main;
      }
      &.btn-group__btn--link {
        color: $color-primary;
      }
    }
  }
}

.avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
</style>
