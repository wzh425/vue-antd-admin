<template>
  <a-modal
    :title="form.id ? '修改文章' : '添加文章'"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
  >
    <a-spin :spinning="confirmLoading">
      <a-form-model
        ref="ruleForm"
        :model="form"
        :rules="rules"
        :label-col="labelCol"
        :wrapper-col="wrapperCol"
      >
        <a-form-model-item label="所属类别" prop="categoryId">
          <a-tree-select
            v-model="form.categoryId"
            style="width: 100%"
            :dropdown-style="{ maxHeight: '400px', overflow: 'auto' }"
            :tree-data="categorys"
            :replaceFields="replaceFields"
            placeholder="请选择"
            tree-default-expand-all
          >
          </a-tree-select>
        </a-form-model-item>
        <a-form-model-item label="标题" ref="title" prop="title">
          <a-input
            v-model="form.title"
            placeholder="请输入标题"
            @blur="
              () => {
                $refs.title.onFieldBlur();
              }
            "
          />
        </a-form-model-item>
        <a-form-model-item label="封面图" prop="imgId">
          <upload-img v-model="form.imgId" :maxCount="1"></upload-img>
        </a-form-model-item>
        <a-form-model-item label="标签" help="多个可用英文逗号分隔开，如：a,b">
          <dynamic-tag v-model="tags" />
        </a-form-model-item>
        <a-form-model-item label="排序" help="数字，越小越向前">
          <a-input-number v-model="form.displayOrder" />
        </a-form-model-item>
        <a-form-model-item label="浏览次数" help="点击浏览该信息自动+1">
          <a-input-number v-model="form.viewCount" />
        </a-form-model-item>
        <a-form-model-item label="发布时间" help="不选择默认当前发布时间">
          <a-date-picker v-model="form.creationTime" show-time />
        </a-form-model-item>
        <a-form-model-item label="内容">
          <quill-editor
          v-model="form.content"
          ref="myQuillEditor"
          :options="editorOption"
        ></quill-editor>
        </a-form-model-item>
      </a-form-model>
    </a-spin>
  </a-modal>
</template>

<script>
import { get,createUpdate } from "@/services/article";
import { getTrees as getCategorys } from "@/services/articleCategory";
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";
import "quill/dist/quill.bubble.css";
import { quillEditor } from "vue-quill-editor";
import quillConfig from "@/utils/quill-config";
import UploadImg from '@/components/uploadImg/UploadImg'
import DynamicTag from '@/components/tag/DynamicTag'
export default {
  name: "ArticleForm",
  components: {
    quillEditor,
    UploadImg,
    DynamicTag
  },
  data() {
    return {
      labelCol: { span: 7 },
      wrapperCol: { span: 10 },
      visible: false,
      confirmLoading: false,
      form: {},
      rules: {
        title: [{ required: true, message: "标题必须填写", trigger: "blur" }],
        // categoryId: [
        //   { required: true, message: "请选择类别", trigger: "change" },
        // ],
      },
      categorys: [],
      replaceFields:{children:'children', title:'displayName', key:'id',value:'id' },
      tags: [],
      inputVisible: false,
      inputValue: '',
      editorOption: quillConfig,
      editor: null,
    };
  },
  beforeDestroy() {
    this.editor = null;
    delete this.editor;
  },
  created() {
    this.getCategorys();
  },
  methods: {
    openModal(model){
      this.visible = true;
      this.form = model;
      if(model && model.id){
        this.getFormData(model.id);
      }
    },
    resetForm() {
      this.form = {};
    },
    getFormData(id) {
      this.confirmLoading = true;
      get(id).then((res) => {
        this.form = {
          ...res,
        };
        if(res.tags){
          this.tags=res.tags.split(",");
        }
      }).finally(()=>{
        this.confirmLoading = false
      });
    },
    getCategorys() {
      getCategorys().then((res) => {
        this.categorys = res;
      });
    },
    handleCancel() {
      this.visible = false
      this.currentStep = 0
    },
    handleOk() {
      const form = this.$refs.ruleForm;
      this.confirmLoading = true;
      form.validate((valid) => {
        if (valid) {
          let values = this.form;
          values.tags=this.tags.toString();
          createUpdate(values)
            .then((res) => {
              this.visible = false;
              form.resetFields();
              this.$message.info("操作成功");
              this.$emit('ok')
            })
            .finally(() => {
              this.confirmLoading = false;
            });
        } else {
          this.confirmLoading = false;
        }
      });
    }
  },
  
};
</script>

<style scoped>
.ql-editor {
  height: 600px !important;
}
.quill_box .ql-container {
  height: 600px !important;
}
</style>