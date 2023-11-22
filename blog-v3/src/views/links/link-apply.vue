<!--友链列表  -->
<script setup>
import { ref, reactive, h } from "vue";
import { addFriendLinks } from "@/api/links";
import { ElNotification } from "element-plus";

import { imgUpload } from "@/api/user";

import Upload from "@/components/Upload/upload.vue";
import router from "@/router";

const urlV = (rule, value, cb) => {
  if (!value) {
    return cb(new Error("请输入网站地址"));
  } else {
    cb();
  }
};

const loading = ref(false);
const activeName = ref("");

const formRef = ref();
const form = reactive({
  site_name: "", // 网站名称
  site_desc: "", // 网站描述
  url: "", // 网址
  site_avatar: "", // 网站头像
  bgList: [],
});
const primaryForm = reactive({ ...form });

const rules = reactive({
  site_name: [{ required: true, message: "请输入网站名称", trigger: "blur" }],
  site_desc: [{ required: true, message: "请输入网站描述", trigger: "blur" }],
  url: [{ required: true, validator: urlV, trigger: "blur" }],
});

// 申请友链
const applayLinks = async () => {
  await formRef.value.validate(async (valid) => {
    if (valid) {
      loading.value = true;
      if (form.bgList.length) {
        const img = await imgUpload(form.bgList[0]);
        if (img.code == 0) {
          const { url } = img.result;
          form.site_avatar = url;
        }
      }
      const res = await addFriendLinks(form);
      if (res && res.code == 0) {
        ElNotification({
          offset: 60,
          title: "提示",
          message: h(
            "div",
            { style: "color: #7ec050; font-weight: 600;" },
            "申请成功，等待博主审核通过"
          ),
        });
        activeName.value = "";
        Object.assign(form, primaryForm);
        router.go(-1);
      } else {
        ElNotification({
          offset: 60,
          title: "错误提示",
          message: h("div", { style: "color: #f56c6c; font-weight: 600;" }, res.message),
        });
      }
      loading.value = false;
    }
  });
};
</script>

<template>
  <PageHeader />
  <div class="center_box flex justify-center flex-col items-center">
    <el-form
      class="apply-form"
      ref="formRef"
      :model="form"
      :rules="rules"
      label-width="100px"
      label-suffix=":"
    >
      <el-form-item label="网站名称" prop="site_name">
        <el-input
          v-model="form.site_name"
          :style="{ width: '220px' }"
          placeholder="请输入网站名称"
          clearable
        />
      </el-form-item>
      <el-form-item label="网站描述" prop="site_desc">
        <el-input
          type="textarea"
          v-model="form.site_desc"
          :style="{ width: '220px' }"
          maxlength="125"
          resize="none"
          :autosize="{ minRows: 2, maxRows: 3 }"
          show-word-limit
          placeholder="请输入网站描述"
          clearable
        />
      </el-form-item>
      <el-form-item label="网站地址" prop="url">
        <el-input
          v-model="form.url"
          :style="{ width: '220px' }"
          placeholder="请输入网站地址"
          clearable
        />
      </el-form-item>
      <el-form-item label="网站头像" prop="site_avatar">
        <Upload
          v-model:file-list="form.bgList"
          :limit="1"
          :width="280"
          :height="140"
          :preview="false"
        />
      </el-form-item>
    </el-form>
    <div class="pos">
      <el-button
        :disabled="loading"
        :loading="loading"
        class="apply-button"
        type="danger"
        @click="applayLinks"
      >
        {{ loading ? "努力上传中..." : "申请友链" }}
      </el-button>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.pos {
  margin-top: 30px;
}
.apply-form {
  max-width: 400px;
}

:deep(.el-upload--picture-card) {
  width: 100px !important;
  height: 100px !important;
  border-radius: 50px !important;
}

:deep(.el-upload-list__item) {
  width: 100px !important;
  height: 100px !important;
  border-radius: 50px !important;
  margin: 0;
}

:deep(.el-upload-list--picture-card) {
  width: 100px !important;
  height: 100px !important;
  border-radius: 50px !important;
}
</style>
