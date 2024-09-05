<template>
  <el-row :gutter="20">
    <el-col :sm="2" class="hidden-xs-only" style="opacity: 0">左侧占位</el-col>
    <el-col :xs="24" :sm="15">
      <el-card
        style="background-color: rgba(255, 255, 255, 0.9)"
        class="left-item"
      >
        <div slot="header" class="total">
          <div class="titleIndex">
            <i v-if="selected" class="el-icon-back" @click="updateBlogList"></i>
            <span>{{ selectMethod }}</span>
          </div>
          <span>
            <el-button
              type="primary"
              plain
              icon="el-icon-plus"
              size="large"
              @click="handleAdd"
              v-hasPermi="['cms:blog:add']"
              >Add</el-button
            ></span
          >
        </div>
        <el-row
          type="flex"
          align="middle"
          style="flex-wrap: wrap"
          :gutter="20"
          v-for="blog in blogList"
          :key="blog.id"
          shadow="never"
          class="blog-content"
        >
          <div @click="getBlogInfo(blog.id)">
            <el-col class="img" :xs="24" :sm="6">
              <el-image
                v-if="blog.blogPicType == '0'"
                lazy
                :src="blog.blogPicLink"
              >
                <div slot="error" class="image-slot">
                  <el-image src="/errorImg.jpg" fit="cover" class="blogPic"
                    >></el-image
                  >
                </div>
              </el-image>
              <el-image
                v-if="blog.blogPicType == '1'"
                lazy
                :src="blog.blogPic"
              ></el-image>
            </el-col>
            <el-col
              :xs="24"
              :sm="18"
              style="
                padding-left: 10px;
                padding-right: 10px;
                margin-bottom: 5px;
                margin-top: -5px;
              "
            >
              <div>
                <h3>
                  <svg-icon icon-class="Topping" v-show="blog.top == 1" />
                  {{ blog.title }}
                </h3>
                <div style="margin-bottom: 10px">
                  <span style="color: rgba(0, 0, 0, 0.4)">
                    {{ blog.blogDesc }}</span
                  >
                </div>
                <div style="margin-bottom: 10px">
                  <el-tag
                    effect="plain"
                    size="mini"
                    v-for="tag in blog.tags"
                    :key="tag.tagId"
                    type="success"
                  >
                    {{ tag.tagName }}
                  </el-tag>
                </div>
                <div class="blog-info">
                  <div class="user-info">
                    <i class="el-icon-user"></i>
                    <span class="header"> {{ blog.createBy }}</span>
                  </div>
                  <div class="blog-date">
                    <i class="el-icon-date"></i>
                    <span> {{ blog.createTime }}</span>
                  </div>
                  <div>
                    <i class="el-icon-view"></i>
                    <span> {{ blog.views }}</span>
                  </div>
                  <div class="blog-type">
                    <el-tag
                      size="mini"
                      v-for="tag in blog.types"
                      :key="tag.typeId"
                      type="info"
                    >
                      {{ tag.typeName }}
                    </el-tag>
                  </div>
                </div>
              </div>
            </el-col>
          </div>
        </el-row>
        <pagination
          v-show="total > 0"
          :total="total"
          :page.sync="queryParams.pageNum"
          :limit.sync="queryParams.pageSize"
          background
          layout="prev, pager, next"
          @pagination="getBlogList"
          style="margin-bottom: 30px; float: right; margin-right: 10px"
        />
      </el-card>
    </el-col>
    <el-col :xs="24" :sm="5">
      <el-card
        style="background-color: rgba(255, 255, 255, 0.9)"
        class="right-item"
      >
        <div slot="header" class="attributes">
          <b>Category</b>
        </div>
        <ul class="blog-type-ul" style="margin-top: 5px">
          <li
            class="blog-type-li"
            v-for="cmsType in typeList"
            :key="cmsType.typeId"
            @click="selectType(cmsType)"
            :class="cmsType.typeId === typeId ? 'activeType' : ''"
          >
            <div style="display: flex; align-items: center">
              <el-image
                style="
                  width: 28px;
                  height: 28px;
                  border-radius: 50%;
                  margin-right: 10px;
                "
                lazy
                :src="cmsType.typePicLink"
                v-show="cmsType.typePicType == '0'"
              >
                <div
                  slot="error"
                  style="width: 28px; height: 28px; border-radius: 50%"
                >
                  <i class="el-icon-collection" style="margin-left: 6px"></i>
                </div>
              </el-image>
              <el-image
                style="
                  width: 28px;
                  height: 28px;
                  border-radius: 50%;
                  margin-right: 10px;
                "
                lazy
                :src="cmsType.typePic"
                v-show="cmsType.typePicType == '1'"
              >
                <div
                  slot="error"
                  style="width: 28px; height: 28px; border-radius: 50%"
                >
                  <i class="el-icon-collection" style="margin-left: 6px"></i>
                </div>
              </el-image>
              {{ cmsType.typeName }}
            </div>
            <div>{{ cmsType.blogNum }}</div>
          </li>
        </ul>
        <div class="more" @click="dealType">
          <i v-if="moreType" class="el-icon-arrow-down"></i>
          <i v-else class="el-icon-arrow-up"></i>
        </div>
      </el-card>
      <el-card
        style="background-color: rgba(255, 255, 255, 0.9)"
        class="right-item"
      >
        <div slot="header" class="attributes">
          <b>Tag</b>
        </div>
        <div class="tags">
          <div
            class="tag-item"
            v-for="tag in tagList"
            :key="tag.tagId"
            @click="selectTag(tag)"
            :class="tag.tagId === tagId ? 'activeTag' : ''"
          >
            <div class="sjx-outer">
              <div class="sjx-inner"></div>
            </div>
            <div class="tag">
              {{ tag.tagName }}
              {{ tag.blogNum }}
            </div>
          </div>
        </div>
        <div class="more" @click="dealTag">
          <i v-if="moreTag" class="el-icon-arrow-down"></i>
          <i v-else class="el-icon-arrow-up"></i>
        </div>
      </el-card>
      <el-card
        style="background-color: rgba(255, 255, 255, 0.9)"
        class="right-item"
      >
        <div slot="header" class="attributes">
          <b>Latest Recommendations</b>
        </div>
        <div
          class="recommend-blog l-text"
          v-for="blog in recommendList"
          :key="blog.id"
          @click="getBlogInfo(blog.id)"
        >
          <a class="recommend-a">{{ blog.title }}</a>
        </div>
      </el-card>
    </el-col>
    <el-col :sm="2" class="hidden-xs-only" style="opacity: 0">右侧占位</el-col>
    <!-- 添加或修改文章管理对话框 -->
    <el-dialog
      :title="title"
      :visible.sync="open"
      :before-close="cancel"
      width="1200px"
      append-to-body
    >
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="Title" prop="title">
          <el-input v-model="form.title" placeholder="please input title" />
        </el-form-item>
        <el-row>
          <el-col :span="8">
            <el-form-item label="Cover">
              <el-radio-group v-model="form.blogPicType">
                <el-radio-button label="0">Url</el-radio-button>
                <el-radio-button label="1">Upload</el-radio-button>
              </el-radio-group>
              <div v-show="form.blogPicType == '0'" class="tabBlock">
                <el-input
                  v-model="form.blogPicLink"
                  placeholder="please input url https://"
                  style="margin-bottom: 10px"
                />
                <el-image
                  :src="form.blogPicLink"
                  :preview-src-list="[form.blogPicLink]"
                  fit="cover"
                  class="blogPic"
                >
                  <div slot="error" class="image-slot">
                    <el-image
                      src="/errorImg.jpg"
                      fit="cover"
                      class="blogPic"
                    ></el-image>
                  </div>
                </el-image>
              </div>
              <div v-show="form.blogPicType == '1'" class="tabBlock">
                <imageUpload v-model="form.blogPic" :limit="1" />
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="16">
            <el-form-item label="About">
              <el-input
                type="textarea"
                v-model="form.blogDesc"
                :autosize="{ minRows: 7, maxRows: 7 }"
                maxlength="50"
                show-word-limit
                placeholder="please input About"
              />
            </el-form-item>
          </el-col>
        </el-row>
        <el-form-item label="content">
          <!-- 图片用base64存储,url方式移动端会显示异常 -->
          <el-row style="margin-bottom: 20px">
            <el-col align="right">
              <span
                v-show="form.contentType === '2'"
                style="color: #e6a23c; margin-right: 20px"
                >Markdown will replace the content</span
              >
              Editor:
              <el-select v-model="form.contentType" placeholder="please select">
                <el-option key="1" label="Quill" value="1" />
                <el-option key="2" label="CherryMarkdown " value="2" />
                <el-option key="3" label="Tinymce" value="3" />
              </el-select>
            </el-col>
          </el-row>
          <el-row>
            <el-col>
              <cmsEditor
                v-if="form.contentType === '1'"
                v-model="form.content"
                @getFileId="getFileId"
                type="base64"
                :min-height="192"
              />
              <CherryMarkdown
                ref="CherryMarkdown"
                v-if="form.contentType === '2'"
                :height="400"
                v-model="form.contentMarkdown"
              ></CherryMarkdown>
              <Tinymce
                v-if="form.contentType === '3'"
                :height="400"
                v-model="form.content"
              ></Tinymce>
            </el-col>
          </el-row>
        </el-form-item>
        <el-form-item label="Tags">
          <el-checkbox-group v-model="form.tagIds">
            <el-checkbox
              v-for="item in tagOptions"
              :label="item.tagId"
              :key="item.tagId"
              :value="item.tagId"
            >
              {{ item.tagName }}
            </el-checkbox>
          </el-checkbox-group>
        </el-form-item>
        <el-row>
          <el-col :span="7">
            <el-form-item label="Category">
              <el-select
                v-model="form.typeIds"
                multiple
                placeholder="please select a category"
                filterable
                clearable
              >
                <el-option
                  v-for="item in typeOptions"
                  :key="item.typeId"
                  :label="item.typeName"
                  :value="item.typeId"
                >
                </el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="17">
            <el-form-item label="Top">
              <el-checkbox v-model="top"></el-checkbox>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="releaseForm">Publish</el-button>
        <el-button @click="cancel">Cancel</el-button>
      </div>
    </el-dialog>
  </el-row>
</template>

<script>
import "element-ui/lib/theme-chalk/display.css";
import CherryMarkdown from "@/components/CherryMarkdown";
import Tinymce from "@/components/Tinymce";
import { Loading } from "element-ui";
import {
  cmsListBlog,
  getBlogDetail,
  cmsListByTypeId,
  cmsListByTagId,
  cmsListRecommend,
  getBlog,
  delBlog,
  updateBlog,
  cancelBlog,
  addBlog,
} from "@/api/cms/blog";
import { addFileBlogInfo } from "@/api/cms/fileBlogInfo";
export default {
  name: "cmsIndex",
  components: {
    CherryMarkdown,
    Tinymce,
  },
  data() {
    return {
      totalcount: 100,
      // 类型选项
      typeOptions: [],
      // 标签选项
      tagOptions: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 8,
      },
      fileIds: [],
      intro: "",
      blogList: [],
      form: {},
      typeList: [],
      tagList: [],
      fullTypeList: [],
      fullTagList: [],
      recommendList: [],
      selectMethod: "All Posts",
      typeId: -1,
      tagId: -1,
      selected: false,
      moreType: true,
      moreTag: true,
      value: new Date(),
      timer: null,
      start: false,
      screenWidth: document.documentElement.clientWidth, //实时屏幕宽度
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        title: null,
        type: 1,
        content: null,
        top: null,
        views: null,
        status: null,
        createBy: null,
      },
      top: false,
      rules: {
        title: [
          {
            required: true,
            message: "Title should not be empty",
            trigger: "blur",
          },
        ],
        type: [
          {
            required: true,
            message: "Type should not be empty",
            trigger: "change",
          },
        ],
      },
      // 总条数
      total: 0,
    };
  },
  computed: {
    pagSmall() {
      return this.screenWidth <= 768;
    },
    // 计算分页栏样式
    pagLayout() {
      if (this.screenWidth < 768) {
        return "prev, pager, next";
      } else {
        return "total, prev, pager, next, jumper";
      }
    },
  },
  created() {
    window.addEventListener("resize", this.screenAdapter);
  },
  mounted() {
    this.$nextTick(function () {
      // 仅在整个视图都被渲染之后才会运行的代码
      this.getTypeList();
      this.getBlogList();
      this.getTagList();
      this.getRecommendList();
      let str =
        "这是我的个人博客、会分享关于编程，开发以及其他方面的一些content，希望可以对您有所帮助...";
      let idx = 0;
      let that = this;
      let timer = setTimeout(function fn() {
        // console.log(this.intro)
        that.intro = that.intro + str.substring(idx, idx + 1);
        idx++;
        if (idx > str.length) {
          that.intro = "";
          idx = 0;
        }
        setTimeout(fn, 200);
      }, 2000);

      this.screenWidth = document.documentElement.clientWidth;
    });
  },
  methods: {
    /** 获取博客列表 */
    getBlogList() {
      let loadingInstance = Loading.service({
        target: ".left-item",
      });
      cmsListBlog(this.queryParams)
        .then((response) => {
          this.blogList = this.picSrc(response.rows);
          this.total = response.total;
        })
        .finally(() => {
          loadingInstance.close();
        });
    },
    //首图地址修改
    picSrc(blogList) {
      for (let i = 0; i < blogList.length; i++) {
        let blogInfo = blogList[i];
        if (blogInfo.blogPic.length > 0) {
          blogList[i].blogPic = process.env.VUE_APP_BASE_API + blogInfo.blogPic;
        } else {
          blogList[i].blogPic = "/errorImg.jpg";
        }
      }
      return blogList;
    },
    // 开始进入主页
    startRead() {
      this.$nextTick(() => {
        document.getElementById("index").scrollIntoView({
          behavior: "smooth",
          block: "start",
          // inline: 'nearest'
        });
      });
    },
    compare(property) {
      return function (a, b) {
        let value1 = a[property].length;
        let value2 = b[property].length;
        return value2 - value1;
      };
    },
    // 获取推荐博客列表
    async getRecommendList() {
      cmsListRecommend(this.queryParams).then((response) => {
        const { data: res } = response;
        this.recommendList = response.rows.slice(0, 4);
        this.total = response.total;
      });
    },
    // 获取博客类型列表
    async getTypeList() {
      getBlogDetail(this.$route.query.id).then((response) => {
        for (let i = 0; i < response.types.length; i++) {
          let typeInfo = response.types[i];
          if (typeInfo.typePic.length > 0) {
            response.types[i].typePic =
              process.env.VUE_APP_BASE_API + typeInfo.typePic;
          }
        }
        const { data: res } = response;
        this.fullTypeList = response.types;
        this.typeList = response.types.slice(0, 4);
      });
    },
    // 获取博客Tag列表
    async getTagList() {
      getBlogDetail(this.$route.query.id).then((response) => {
        const { data: res } = response;
        this.fullTagList = response.tags;
        this.tagList = response.tags.slice(0, 6);
      });
    },
    /** 新增按钮操作 */
    handleAdd() {
      getBlog().then((response) => {
        this.typeOptions = response.types;
        this.tagOptions = response.tags;
        this.reset();
        this.open = true;
        this.title = "Add";
      });
    },
    getFileId(data) {
      this.fileIds.push(data);
    },
    // 取消按钮
    cancel() {
      let fileids = this.fileIds;
      if (fileids.length > 0) {
        delFileInfo(fileids);
      }
      this.fileIds.length = 0;
      cancelBlog(this.form).then((response) => {});
      this.top = false;
      this.open = false;
      this.blogFilesOpen = false;
      this.reset();
    },
    setFormContent() {
      this.form.content = this.$refs.CherryMarkdown.getCherryHtml();
    },
    /** 发布按钮 */
    releaseForm() {
      this.$refs["form"].validate((valid) => {
        if (valid) {
          this.$modal
            .confirm("Whether to confirm the release?")
            .then(() => {
              this.form.type = 1;
              this.form.status = 1;
              if (this.top) {
                this.form.top = 1;
              } else {
                this.form.top = 0;
              }
              if (this.form.contentType === "2") {
                this.setFormContent();
              }
              if (this.form.id != null) {
                updateBlog(this.form).then((response) => {
                  if (this.fileIds.length > 0) {
                    let fileBlogInfo = {
                      blogId: this.form.id,
                      fileIds: this.fileIds,
                    };
                    addFileBlogInfo(fileBlogInfo).then((response) => {});
                  }
                  this.$modal.msgSuccess("Success");
                  this.fileIds.length = 0;
                  this.open = false;
                  this.getList();
                });
              } else {
                addBlog(this.form).then((response) => {
                  if (this.fileIds.length > 0) {
                    let fileBlogInfo = {
                      blogId: response.data,
                      fileIds: this.fileIds,
                    };
                    addFileBlogInfo(fileBlogInfo).then((response) => {});
                  }
                  this.$modal.msgSuccess("Success");
                  this.fileIds.length = 0;
                  this.open = false;
                  this.getList();
                });
              }
            })
            .catch(() => {});
        }
      });
    },
    // 表单重置
    reset() {
      this.form = {
        id: null,
        createBy: null,
        createTime: null,
        updateBy: null,
        updateTime: null,
        title: null,
        type: 1,
        content: null,
        top: "0",
        views: null,
        status: "0",
        blogDesc: null,
        blogFiles: null,
        blogPicType: "0",
        blogPic: null,
        blogPicLink: null,
        tagIds: [],
        typeIds: [],
        blogFilesNew: [],
        contentType: "1",
        contentMarkdown: null,
      };
      this.resetForm("form");
    },
    // 跳转到博客详情页
    getBlogInfo(blogId) {
      let routeUrl = this.$router.resolve({
        path: "/cms/main/blog",
        query: {
          id: blogId,
        },
      });
      window.open(routeUrl.href, "_blank");
    },
    // 修改当前页码
    handleCurrentChange(newSize) {
      this.queryInfo.pagenum = newSize;
      this.getBlogList();
    },
    // 修改当前页大小
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
    },
    // 按Category筛选博客
    async selectType(cmsType) {
      let loadingInstance = Loading.service({
        target: ".left-item",
      });
      this.typeId = cmsType.typeId;
      cmsListByTypeId(this.typeId)
        .then((response) => {
          this.blogList = this.picSrc(response.rows);
          this.total = response.total;
          // this.totalcount = res.data.totalElements
          this.selectMethod = "Category: " + cmsType.typeName;
          this.selected = true;
        })
        .finally(() => {
          loadingInstance.close();
        });
    },
    // 按Tag筛选博客
    async selectTag(tag) {
      let loadingInstance = Loading.service({
        target: ".left-item",
      });
      this.tagId = tag.tagId;
      cmsListByTagId(this.tagId)
        .then((response) => {
          this.blogList = this.picSrc(response.rows);
          this.total = response.total;
          // this.totalcount = res.data.totalElements
          this.selectMethod = "Tag: " + tag.tagName;
          this.selected = true;
        })
        .finally(() => {
          loadingInstance.close();
        });
    },
    // 更新博客列表
    updateBlogList() {
      this.selected = false;
      this.typeId = -1;
      this.tagId = -1;
      this.selectMethod = "All Posts";
      this.getBlogList();
    },
    // 得到所有的Tag
    async getFullTagList() {
      this.tagList = this.fullTagList;
    },
    async dealType() {
      if (this.moreType) {
        this.typeList = this.fullTypeList;
      } else {
        this.typeList = this.fullTypeList.slice(0, 4);
      }
      this.moreType = !this.moreType;
    },
    async dealTag() {
      if (this.moreTag) {
        await this.getFullTagList();
      } else {
        this.tagList = this.fullTagList.slice(0, 6);
      }
      this.moreTag = !this.moreTag;
    },
    // 屏幕尺寸变化的监听函数
    screenAdapter() {
      this.screenWidth = document.documentElement.clientWidth;
    },
  },
};
</script>

<style scoped>
.welcome {
  background-color: rgba(0, 0, 0, 0.1);
  border: none;
  height: 90%;
  position: relative;
}

.border {
  width: 812px;
  height: 112px;
  position: absolute;
  top: -6px;
  left: -6px;
  border: 3px solid white;
  box-sizing: border-box;
  animation: clipMe 5s linear infinite;
}

.tit {
  box-sizing: border-box;
  position: relative;
  width: 800px;
  height: 100px;
  line-height: 100px;
  box-shadow: inset 0 0 0 1px white;
  margin: 40px auto;
  margin-top: 80px;
  color: white;
  text-align: center;
  font-size: 50px;
  font-weight: normal;
  letter-spacing: 10px;
}

.intro {
  letter-spacing: 5px;
  line-height: 50px;
  width: 80%;
  margin: 0 auto;
  text-align: center;
  font-weight: normal;
  color: white;
}

.down {
  animation: bounce 2s infinite;
  animation-duration: 3s;
  font-size: 25px;
  position: absolute;
  bottom: 5px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  justify-content: center;
  align-items: center;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  border: 2px solid #fff;
}

.down:hover {
  animation: none;
  cursor: pointer;
  box-shadow: 0 0 20px 0 white;
  transition: all 0.2s;
}

.left-item .pagination-container {
  background: rgb(255, 255, 255, 0);
}

@keyframes clipMe {
  0%,
  100% {
    clip: rect(0px, 806px, 6px, 0px);
  }

  25% {
    clip: rect(0px, 6px, 112px, 0px);
  }

  50% {
    clip: rect(112px, 812px, 112px, 0px);
  }

  75% {
    clip: rect(0px, 812px, 112px, 806px);
  }
}

@keyframes bounce {
  0%,
  20%,
  50%,
  80%,
  100% {
    transform: translate(-50%, 0);
  }

  40% {
    transform: translate(-50%, -30px);
  }

  60% {
    transform: translate(-50%, -15px);
  }
}

.blog-type-ul {
  padding-left: 10px;
  padding-right: 10px;
  margin-bottom: 0;
  border-radius: 5px;
}

.el-pagination {
  padding-bottom: 20px;
}

.el-card /deep/ .el-card__body {
  padding: 0;
}

.right-item {
  margin-bottom: 20px;
}

.blog-type-li:first-child {
  border-top: 1px solid rgba(179, 216, 255, 0.5);
}

.blog-type-li {
  border-bottom: 1px solid rgba(179, 216, 255, 0.5);
}

.more {
  text-align: center;
  color: #3a8ee6;
  padding: 8px;
}

.more:hover {
  cursor: pointer;
  color: #3a8ee6;
}

.blog-type-li:hover {
  background-color: rgba(213, 255, 255, 0.3);
  cursor: pointer;
}

.activeType {
  background-color: rgba(58, 142, 230, 0.3);
  cursor: pointer;
}

.tags {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  margin: 15px 13px 0;
  border-bottom: 1px solid rgba(179, 216, 255, 0.5);
}

.tag-item {
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-left: 5px;
  margin-right: 5px;
  margin-bottom: 10px;
  box-sizing: border-box;
}

.tag {
  background-color: #ecf5ff;
  box-sizing: border-box;
  display: inline-block;
  height: 22px;
  padding: 0 10px;
  line-height: 22px;
  font-size: 10px;
  color: #409eff;
  border-radius: 4px;
  white-space: nowrap;
  border: 1px solid #409eff;
  transition: 0.2s;
}

.sjx-outer {
  width: 0;
  height: 0;
  border-top: 6px solid transparent;
  border-bottom: 6px solid transparent;
  border-right: 6px solid #409eff;
  position: relative;
  transition: 0.2s;
}

.sjx-inner {
  border-top: 6px solid transparent;
  border-bottom: 6px solid transparent;
  border-right: 6px solid #ecf5ff;
  top: -6px;
  left: 1px;
  position: absolute;
  transition: 0.2s;
}

.tag-item:hover,
.activeTag {
  box-sizing: border-box;
}

.tag {
  color: white;
  background-color: #409eff;
  cursor: pointer;
}

.sjx-inner {
  border-right: 6px solid #409eff;
}

.blog-type-li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  line-height: 40px;
}

.recommend-blog {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  padding-left: 10px;
  padding-right: 10px;
  margin-bottom: 0;
  border-radius: 5px;
}

.recommend-a {
  border-bottom: 1px solid rgba(34, 36, 38, 0.15);
  line-height: 40px;
  display: block;
  text-decoration: none;
  color: black;
}

.recommend-a:hover {
  color: #3a8ee6;
}

.total {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: larger;
  font-weight: bold;
}

.titleIndex {
  display: flex;
  align-items: center;
}

.el-icon-back {
  font-weight: bolder;
  color: #3a8ee6;
  margin-right: 10px;
}

.el-icon-back:hover {
  cursor: pointer;
}

.blog-content:hover {
  border-left: 5px solid #3a8ee6;
  border-right: 5px solid #3a8ee6;
  background-color: rgba(58, 142, 230, 0.3);
  cursor: pointer;
}

.blog-content {
  padding: 10px;
  height: auto;
  border-bottom: 1px solid rgb(199, 163, 92);
  /*border-bottom: 1px solid rgba(34, 36, 38, .15);*/
  transition: 0.3s;
}

.el-image {
  border-radius: 5px;
  box-sizing: border-box;
  flex-shrink: 0;
}

.blog-info {
  display: flex;
  align-items: center;
  color: rgba(0, 0, 0, 0.4);
  font-size: 10px;
}

.user-info {
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-right: 15px;
  float: left;
}

.header {
  text-decoration: none;
  color: #3a8ee6;
  font-weight: bold;
}

.blog-date {
  float: right;
  margin-right: 15px;
}

.blog-type {
  float: right;
  margin-left: auto;
}

.blog-tag {
  float: right;
  margin-left: auto;
}

@media screen and (max-width: 768px) {
  .blog-date {
    display: none;
  }

  .welcome {
    width: 100%;
  }

  .border {
    display: none;
  }

  .tit {
    font-size: 2rem;
    width: 100%;
    line-height: 50px;
    letter-spacing: 2px;
    height: auto;
  }

  .intro {
    font-size: 1rem;
    line-height: 30px;
  }

  .el-pagination {
    width: 100%;
  }
}
</style>
