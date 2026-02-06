<template>
  <div class="addModelContainer">
    <a-modal v-model:open="modelShow" :title="props.state" :footer="null" width="520px">
      <a-form :model="modelForm">
        <a-form-item label="名称">
          <a-input v-model:value="modelForm.name" />
        </a-form-item>
        <a-form-item label="厂商">
          <a-select placeholder="请选择厂商" v-model:value="modelForm.manufacturer" style="width: 100%">
            <a-select-option value="deepSeek">DeepSeek</a-select-option>
            <a-select-option value="volcengine">火山引擎</a-select-option>
            <a-select-option value="kling">可灵</a-select-option>
            <a-select-option value="zhipu">智谱</a-select-option>
            <a-select-option value="qwen">通义千问</a-select-option>
            <a-select-option value="openai">OpenAI</a-select-option>
            <a-select-option value="vidu">Vidu</a-select-option>
            <a-select-option value="runninghub">RunningHUB</a-select-option>
            <a-select-option value="other">其他</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item label="类型">
          <a-select
            placeholder="请选择类型"
            ref="select"
            style="width: 100%"
            :allowClear="true"
            :options="typeOptions"
            :field-names="{ label: 'name', value: 'value', options: 'children' }"
            v-model:value="modelForm.type" />
        </a-form-item>

        <a-form-item label="模型">
          <a-select
            v-if="modelForm.manufacturer !== 'other'"
            placeholder="请选择模型"
            ref="select"
            style="width: 100%"
            :allowClear="true"
            :options="currentModelPresets"
            :field-names="{ label: 'label', value: 'value' }"
            v-model:value="modelForm.model" />
          <a-input v-else v-model:value="modelForm.model" placeholder="请输入自定义模型标识" />
        </a-form-item>
        <a-form-item label="Base URL">
          <a-input v-model:value="modelForm.baseUrl" />
        </a-form-item>
        <a-form-item label="API Key">
          <a-input v-model:value="modelForm.apiKey" />
        </a-form-item>
        <a-form-item style="text-align: right; margin-bottom: 0">
          <a-button style="margin-right: 8px" @click="modelShow = false">取消</a-button>
          <a-button type="primary" @click="keep">保存</a-button>
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from "vue";
import axios from "@/utils/axios";
import { message } from "ant-design-vue";
import type { SelectProps } from "ant-design-vue";

const modelShow = defineModel<boolean>("modelShow", {
  type: Boolean,
  required: true,
});
const modelForm = defineModel<RowData>("modelForm", {
  type: Object,
  required: true,
});
const props = defineProps({
  state: {
    type: String,
  },
});

interface RowData {
  id: number;
  name: string;
  type: string;
  model: string;
  baseUrl: string;
  manufacturer: string;
  createTime: number;
  apiKey: string;
}
//模型类型选项
const typeOptions = ref<SelectProps["options"]>([
  {
    name: "文本模型",
    children: [
      {
        value: "usuallyText",
        name: "普通",
      },
      {
        value: "deepThinkingText",
        name: "深度思考",
      },
    ],
  },
  {
    name: "图片模型",
    children: [
      {
        value: "i2i",
        name: "图生图",
      },
      {
        value: "t2i",
        name: "文生图",
      },
    ],
  },
  {
    name: "视频模型",
    children: [
      {
        value: "singleImage",
        name: "单图生视频",
      },
      {
        value: "startEndRequired",
        name: "首尾帧（两张都得有）",
      },
      {
        value: "endFrameOptional",
        name: "首尾帧（尾帧可选）",
      },
      {
        value: "startFrameOptional",
        name: "首尾帧（首帧可选）",
      },
      {
        value: "multiImage",
        name: "多图生视频",
      },
      {
        value: "reference",
        name: "参考图生视频",
      },
      {
        value: "text",
        name: "文本生视频",
      },
    ],
  },
]);
//文本模型预设
const textModelPresets = {
  deepSeek: {
    usuallyText: [{ label: "deepseek-chat", value: "deepseek-chat" }],
    deepThinkingText: [{ label: "deepseek-reasoner", value: "deepseek-reasoner" }],
  },
  volcengine: {
    usuallyText: [
      { label: "doubao-lite-4-chat", value: "doubao-lite-4-chat" },
      { label: "doubao-pro-4-chat", value: "doubao-pro-4-chat" },
    ],
    deepThinkingText: [
      { label: "doubao-seed-1-8", value: "doubao-seed-1-8" },
      { label: "doubao-seed-1-6", value: "doubao-seed-1-6" },
      { label: "doubao-seed-1-6-lite", value: "doubao-seed-1-6-lite" },
      { label: "doubao-seed-1-6-flash", value: "doubao-seed-1-6-flash" },
    ],
  },
  zhipu: {
    usuallyText: [
      { label: "glm-4.7", value: "glm-4.7" },
      { label: "glm-4.7-flashx", value: "glm-4.7-flashx" },
      { label: "glm-4.6", value: "glm-4.6" },
      { label: "glm-4.5-air", value: "glm-4.5-air" },
      { label: "glm-4.5-airx", value: "glm-4.5-airx" },
      { label: "glm-4-long", value: "glm-4-long" },
      { label: "glm-4-flashx-250414", value: "glm-4-flashx-250414" },
      { label: "glm-4.7-flash", value: "glm-4.7-flash" },
      { label: "glm-4-flash-250414", value: "glm-4-flash-250414" },
    ],
    deepThinkingText: [
      { label: "glm-4.5-flash", value: "glm-4.5-flash" },
      { label: "glm-4.6v", value: "glm-4.6v" },
    ],
  },
  qwen: {
    usuallyText: [
      { label: "qwen-vl-max", value: "qwen-vl-max" },
      { label: "qwen-plus-latest", value: "qwen-plus-latest" },
      { label: "qwen-max", value: "qwen-max" },
      { label: "qwen2.5-72b-instruct", value: "qwen2.5-72b-instruct" },
      { label: "qwen2.5-14b-instruct-1m", value: "qwen2.5-14b-instruct-1m" },
      { label: "qwen2.5-vl-72b-instruct", value: "qwen2.5-vl-72b-instruct" },
    ],
    deepThinkingText: [],
  },
  openai: {
    usuallyText: [
      { label: "gpt-4o", value: "gpt-4o" },
      { label: "gpt-4o-mini", value: "gpt-4o-mini" },
      { label: "gpt-4.1", value: "gpt-4.1" },
      { label: "gpt-5.1", value: "gpt-5.1" },
      { label: "gpt-5.2", value: "gpt-5.2" },
    ],
    deepThinkingText: [],
  },
  google: {
    usuallyText: [
      { label: "gemini-2.0-flash", value: "gemini-2.0-flash" },
      { label: "gemini-2.0-flash-lite", value: "gemini-2.0-flash-lite" },
      { label: "gemini-1.5-pro", value: "gemini-1.5-pro" },
      { label: "gemini-1.5-flash", value: "gemini-1.5-flash" },
    ],
    deepThinkingText: [
      { label: "gemini-2.5-pro", value: "gemini-2.5-pro" },
      { label: "gemini-2.5-flash", value: "gemini-2.5-flash" },
    ],
  },
  anthropic: {
    usuallyText: [
      { label: "claude-opus-4-5", value: "claude-opus-4-5" },
      { label: "claude-haiku-4-5", value: "claude-haiku-4-5" },
      { label: "claude-sonnet-4-5", value: "claude-sonnet-4-5" },
      { label: "claude-opus-4-1", value: "claude-opus-4-1" },
      { label: "claude-opus-4-0", value: "claude-opus-4-0" },
      { label: "claude-sonnet-4-0", value: "claude-sonnet-4-0" },
      { label: "claude-3-7-sonnet-latest", value: "claude-3-7-sonnet-latest" },
      { label: "claude-3-5-haiku-latest", value: "claude-3-5-haiku-latest" },
    ],
    deepThinkingText: [],
  },
};
//图片模型预设
const imageModelPresets = {
  volcengine: {
    t2i: [{ label: "doubao-seedream-4-5-251128", value: "doubao-seedream-4-5-251128" }],
    i2i: [{ label: "doubao-seedream-4-5-251128", value: "doubao-seedream-4-5-251128" }],
  },
  kling: {
    t2i: [
      { label: "kling-v1", value: "kling-v1" },
      { label: "kling-v1-5", value: "kling-v1-5" },
      { label: "kling-v2", value: "kling-v2" },
      { label: "kling-v2-new", value: "kling-v2-new" },
      { label: "kling-v2-1", value: "kling-v2-1" },
    ],
    i2i: [
      { label: "kling-v1", value: "kling-v1" },
      { label: "kling-v1-5", value: "kling-v1-5" },
      { label: "kling-v2", value: "kling-v2" },
      { label: "kling-v2-new", value: "kling-v2-new" },
      { label: "kling-v2-1", value: "kling-v2-1" },
    ],
  },
  google: {
    t2i: [
      { label: "gemini-2.5-flash-image", value: "gemini-2.5-flash-image" },
      { label: "gemini-3-pro-image-preview", value: "gemini-3-pro-image-preview" },
    ],
    i2i: [
      { label: "gemini-2.5-flash-image", value: "gemini-2.5-flash-image" },
      { label: "gemini-3-pro-image-preview", value: "gemini-3-pro-image-preview" },
    ],
  },
  vidu: {
    t2i: [{ label: "viduq2", value: "viduq2" }],
    i2i: [{ label: "viduq2", value: "viduq2" }],
  },
  runninghub: {
    t2i: [{ label: "nanobanana", value: "nanobanana" }],
    i2i: [{ label: "nanobanana", value: "nanobanana" }],
  },
  apimart: {
    t2i: [{ label: "nanobanana", value: "nanobanana" }],
    i2i: [{ label: "nanobanana", value: "nanobanana" }],
  },
};
//视频模型预设
const videoModelPresets = {
  volcengine: {
    singleImage: [{ label: "doubao-seedance-1-0-pro-fast-251015", value: "doubao-seedance-1-0-pro-fast-251015" }],
    startEndRequired: [],
    endFrameOptional: [
      { label: "doubao-seedance-1-5-pro-251215", value: "doubao-seedance-1-5-pro-251215" },
      { label: "doubao-seedance-1-0-pro-250528", value: "doubao-seedance-1-0-pro-250528" },
      { label: "doubao-seedance-1-0-lite-i2v-250428", value: "doubao-seedance-1-0-lite-i2v-250428" },
    ],
    startFrameOptional: [],
    multiImage: [],
    reference: [{ label: "doubao-seedance-1-0-lite-i2v-250428", value: "doubao-seedance-1-0-lite-i2v-250428" }],
    text: [
      { label: "doubao-seedance-1-0-lite-t2v-250428", value: "doubao-seedance-1-0-lite-t2v-250428" },
      { label: "doubao-seedance-1-0-pro-fast-251015", value: "doubao-seedance-1-0-pro-fast-251015" },
      { label: "doubao-seedance-1-0-pro-250528", value: "doubao-seedance-1-0-pro-250528" },
      { label: "doubao-seedance-1-5-pro-251215", value: "doubao-seedance-1-5-pro-251215" },
    ],
  },
  sora: {
    singleImage: [
      { label: "sora-2", value: "sora-2" },
      { label: "sora-2-pro", value: "sora-2-pro" },
    ],
    startEndRequired: [],
    endFrameOptional: [],
    startFrameOptional: [],
    multiImage: [],
    reference: [],
    text: [],
  },
  kling: {
    singleImage: [
      { label: "kling-v1-6(std)", value: "kling-v1-6" },
      { label: "kling-v1-6(pro)", value: "kling-v1-6" },
    ],
    startEndRequired: [],
    endFrameOptional: [],
    startFrameOptional: [],
    multiImage: [],
    reference: [],
    text: [],
  },
};

const currentModelPresets = computed(() => {
  const { manufacturer, type } = modelForm.value;

  if (manufacturer === "other" || !manufacturer) {
    return [];
  }

  if (!type) {
    return [];
  }
  const textTypes = ["usuallyText", "deepThinkingText"];
  const imageTypes = ["t2i", "i2i"];
  const videoTypes = ["singleImage", "startEndRequired", "endFrameOptional", "startFrameOptional", "multiImage", "reference", "text"];
  let presets: Record<string, any> = {};

  if (textTypes.includes(type)) {
    presets = textModelPresets[manufacturer as keyof typeof textModelPresets] || {};
  } else if (imageTypes.includes(type)) {
    presets = imageModelPresets[manufacturer as keyof typeof imageModelPresets] || {};
  } else if (videoTypes.includes(type)) {
    presets = videoModelPresets[manufacturer as keyof typeof videoModelPresets] || {};
  }

  return presets[type as keyof typeof presets] || [];
});
const emit = defineEmits(["fetchModelList"]);
//保存模型
async function keep() {
  if (modelForm.value.id == 0) {
    try {
      await axios.post("/setting/addModel", {
        name: modelForm.value.name,
        type: modelForm.value.type,
        model: modelForm.value.model,
        baseUrl: modelForm.value.baseUrl,
        manufacturer: modelForm.value.manufacturer,
        apiKey: modelForm.value.apiKey,
      });
      message.success("新增成功");
      emit("fetchModelList");
    } catch (e) {
      message.error("新增失败");
    }
  } else {
    try {
      await axios.post("/setting/updeteModel", {
        id: modelForm.value.id,
        name: modelForm.value.name,
        type: modelForm.value.type,
        model: modelForm.value.model,
        baseUrl: modelForm.value.baseUrl,
        manufacturer: modelForm.value.manufacturer,
        apiKey: modelForm.value.apiKey,
      });
      message.success("编辑成功");
      emit("fetchModelList");
    } catch (e) {
      message.error("编辑失败");
    }
  }

  modelShow.value = false; //关闭弹窗
}
</script>

<style lang="scss" scoped></style>
